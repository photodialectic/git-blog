# Creating a Blog API using Git as the Datastore

Some time ago, I became interested in the concept of using [git](https://en.wikipedia.org/wiki/Git) as a datastore/CMS for a blog. I found the idea of using branching and merging for drafts, as well as versioning for posts, to be intriguing. Additionally, I wanted to test my ability to create a basic API that could be utilized for interacting with the blog. This is the outcome of my experiment.

## Tech stack

- [Python](https://www.python.org/)
- [Torando](https://www.tornadoweb.org/en/stable/)
- [Pygit2](https://www.pygit2.org/)

Python is the language I feel most comfortable with and Tornado is a web framework that I have used in the past. Pygit2 is a Python binding for libgit2, which is a C implementation of git.

### Dockerfile

I have the following Dockerfile to build the image - It will install pygit2 and checkout the repo to `/content`.

```language-bash
FROM python:3.11

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libgit2-dev \
    && rm -rf /var/lib/apt/lists/*

RUN pip install pygit2

COPY ./requirements.txt /app/requirements.txt
RUN pip install -r /app/requirements.txt

COPY . /app

RUN git clone https://github.com/photodialectic/git-blog.git /content

WORKDIR /app

CMD ["python", "/app/service.py"]
```

## API Design

I wanted to keep the API straightfoward so there are really just a few [endpoints](https://www.nickhedberg.com/docs/oas/blog-api.yml)

- `GET /v1/branches` - List all branches
- `GET /v1/entries` - List all entries (essentially the file system)
- `GET /v1/entries/:path` - The path could resolve to either a tree or blob. If its a blob it'll render the contents

### API Utils

I'm used to the idea of just `git checkout <branch>` and then `git pull`. I didn't find this to be as straightfoward with pygit2. So I had to write some utility functions to help me with this.

#### `get_repo`

I wanted to centralize the process of getting the repository to ensure that I set a user. This comes in handy when we pull down a branch.

```language-python
def get_repo(self):
    repo = pygit2.Repository("/content")
    repo.config.set_multivar("user.name", "nh", "nh")
    repo.config.set_multivar("user.email", "info@nh", "nh")
    return repo
```

#### `local_branch`

In order to pull down a branch, we need a local branch (ideally with the same name) to merge changes into. Without this, I found that changes will be merged into the main branch, which is not desired. We want to be able to switch between branches.

```language-python
def local_branch(self, repo, branch_name, remote_branch):
    local_branch = None
    commit = repo.get(remote_branch.target)  # Resolve the Oid to a Commit
    try:
        local_branch = repo.lookup_branch(branch_name, pygit2.GIT_BRANCH_LOCAL)
    except KeyError:
        local_branch = repo.create_branch(branch_name, commit)

    if not local_branch:
        local_branch = repo.create_branch(branch_name, commit)

    repo.set_head(local_branch.name)
    return local_branch
```

#### `pull`

This function is responsible for pulling down changes from a remote branch. It will establish a local branch, check if the branch is up to date, fast-forward the branch if possible, or merge changes if necessary. If there are conflicts, it will return a 409 status code.

```language-python
def pull(self, repo, branch):
    remote_name = "origin"
    remote_branch_ref = f"refs/remotes/{remote_name}/{branch}"
    remote = repo.remotes[remote_name]
    logging.info(f"feteching {remote_name}/{branch}")
    remote.fetch()
    if remote_branch_ref not in repo.references:
        logging.error(f"Branch {branch} does not exist in the remote repository")
        return

    remote_branch = repo.lookup_reference(remote_branch_ref)
    try:
        logging.info("establishing local branch")
        local_branch = self.local_branch(repo, branch, remote_branch)
        if not local_branch:
            raise Exception(f"Could not establish local branch {branch}")

        merge_base = repo.merge_base(repo.head.target, remote_branch.target)

        logging.info(f"pulling remote changes to local branch {branch}")
        merge_result, _ = repo.merge_analysis(remote_branch.target)

        if merge_result & pygit2.GIT_MERGE_ANALYSIS_UP_TO_DATE:
            logging.info("Already up to date")
        elif merge_result & pygit2.GIT_MERGE_ANALYSIS_FASTFORWARD:
            logging.info("Fast-forward merge")
            repo.checkout_tree(repo.get(remote_branch.target))
            repo.head.set_target(remote_branch.target)
        else:
            logging.info("Merging changes")
            repo.merge(remote_branch.target)

            if repo.index.conflicts:
                logging.error("Merge conflicts encountered")
                self.set_status(409)
                self.write("conflict")
                return

            user = repo.default_signature
            tree = repo.index.write_tree()

            commit = repo.create_commit(
                "HEAD",
                user,
                user,
                "Merged changes",
                tree,
                [repo.head.target, remote_branch.target],
            )
            logging.info(f"Merge completed: new commit {commit}")

        # Cleanup merge state
        repo.state_cleanup()

    except Exception as e:
        logging.error(f"An error occurred during pull: {str(e)}")
        self.set_status(500)  # Assuming this is a web framework-like setting
        self.write("Error during pull")
```

#### `get_tree_structure`

This function will return the tree structure of the repository. It will recursively call itself to get the tree structure of any subdirectories.

```language-python
def get_tree_structure(self, repo, tree):
    files = []
    for entry in tree:
        if entry.type_str == "tree":
            files.append(
                {
                    "name": entry.name,
                    "type": "dir",
                    "children": self.get_tree_structure(repo, repo[entry.id]),
                }
            )
        else:
            files.append({"name": entry.name, "type": "file"})
    return files
```

#### `get_tree_from_path`

This function will return the tree structure of a specific path. It will return the tree structure of the repository if the path is empty. If the path is not empty, it will traverse the tree structure to find the path.

```language-python
def get_tree_from_path(self, repo, path):
    tree = repo.get(repo.head.target).tree
    for part in path.split("/"):
        for entry in tree:
            if entry.name == part and entry.type_str == "blob":
                return entry

            if entry.name == part and entry.type_str == "tree":
                tree = repo[entry.id]
                break

    if path.split("/")[-1] != entry.name:
        return None

    return tree
```

## Conclusion

This post is written and served via the Blog API it descibes. That's all for now.
