# [Chores: Family Task Management System](/chores)

A NextJS/React application designed to gamify household chores and manage family responsibilities with goal tracking, approval workflows, and automated rotation systems.

## Overview

I built this so my kids could save up for a Switch2 by performing chores/tasks around the house. After they earned the Switch2, we continued to use it as a family. Anyone can create a family, invite members via QR code, and start assigning chores with photo submission and approval workflows. The app includes gamification elements like progress tracking, achievements, and statistics to keep everyone motivated.

The main idea is the kids will submit chores for approval which helps develop skills like accountibility and determining a value for their work. It very deliberately does not allow parents to assign chores or reward points directly. The kids have to decide what chores they want to do and submit them for approval. This helps them learn to manage their own responsibilities.

## Key Features

### Task Management System
- **Chore Submission**: Kids can submit chores with a point value for approval
- **Approval Workflow**: Parents can approve or reject submitted chores, edit point values, and provide feedback
- **Goal Integration**: Link chores to broader family goals and objectives

### Family Organization
- **Multi-User Support**: Separate profiles for each family member
- **Role-Based Access**: Different permissions for parents and children
- **Family Invitation**: QR code-based family joining system (code can be rotated for security)

### Gamification Elements
- **Progress Tracking**: Visual progress bars for individual and family goals
- **Goal Management**: Create and track both short-term and long-term objectives

## Technical Architecture

### Component Structure
```
components/
├── Avatar.js           # User profile pictures and display
├── PageHeader.js       # Consistent page navigation and branding
├── QRCodeDisplay.js    # Family invitation QR code generation
├── ThemeProvider.js    # Dark/light mode theme management
└── ThemeToggle.js      # Theme switching controls
```

### Database Design
The application uses a MySQL backend with tables for:
- **Users**: Family member profiles and authentication data
- **Goals**: Family objectives and individual targets
- **Chores**: Task definitions and assignment rules
- **Submissions**: Completed chore submissions with photo evidence
- **Family Relationships**: Member roles and permissions

### Authentication & Security
- **Auth0 Integration**: Secure user authentication and session management
- **Profile Management**: User profile creation and family association
- **Image Upload**: Secure photo submission and storage system
- **Access Control**: Role-based permissions for different family members

## Implementation Details

### Family Setup Flow
1. **Family Creation**: Initial parent creates family group
2. **QR Code Generation**: Unique family invitation code created
3. **Member Joining**: Family members scan QR code to join
4. **Profile Setup**: Individual profile configuration and role assignment

## User Experience Design

### Visual Design System
- **Theme Support**: Light and dark mode options
- **Accessible Design**: High contrast ratios and screen reader support
- **Responsive Layout**: Seamless experience across device sizes
- **Visual Feedback**: Clear indicators for completed, pending, and overdue tasks
