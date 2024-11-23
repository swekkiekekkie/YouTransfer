# Documentation Structure

The documentation is organized into several key sections, each serving a specific purpose in the development lifecycle:

### Project Overview

- **Purpose**: Provide a high-level overview of the project, including its purpose, scope, and key stakeholders.
- **Contents**: Brief introduction to the project, its goals, and its target audience.
- **Location**: [`docs/project_overview.md`](docs/project_overview.md)

### User Stories

- **Purpose**: Capture end-user actions, goals, reasoning, and motives to guide development.
- **Contents**: Each user story follows the format: "As a user, I want [an action] so that [a benefit]."
- **Location**: [`docs/user_stories.md`](docs/user_stories.md)

### Functional Requirements

- **Purpose**: Specify the functionalities the system must provide.
- **Contents**: Clear, numbered requirements describing what the system should do.
- **Location**: [`docs/requirements/functional_requirements.md`](docs/requirements/functional_requirements.md)

### Non-Functional Requirements

- **Purpose**: Define system attributes like performance, usability, and maintainability.
- **Contents**: Descriptions of system qualities with measurable criteria. Such as:
    - *Performance*: The system should efficiently handle the expected load.
    - *Usability*: The system should be intuitive and easy to navigate.
    - *Maintainability*: The system should be straightforward to update and maintain.
- **Location**: [`docs/requirements/nonfunctional_requirements.md`](docs/requirements/nonfunctional_requirements.md)

### Design Documents

- **Purpose**: Plan the system design, which includes both technical architecture and user interface design.
- **Contents**:
  - *Technical design*, which gives a high-level overview of the system.
  - *User Interface Design*, which details the UI components, interactions, styling, and technical implementation including:
    - Component architecture and hierarchy
    - State management and data flow
    - Event handling and user interactions
- **Location**:<br>
`docs/design/`<br>
`├── `[`ui_design.md`](docs/design/ui_design.md)<br>
`└── `[`technical_design.md`](docs/design/technical_design.md)

### Prototypes

- **Purpose**: Demonstrate and validate design concepts before full implementation.
- **Contents**: Interactive mockups, wireframes, and proof-of-concept implementations.
- **Location**: [`docs/prototypes/`](docs/prototypes)

### Implementation Plans

- **Purpose**: Detail how the design will be implemented. Technical design covers most of this, so its more about planning and organizing the development process.
- **Contents**: Milestones, tasks, dependencies, and resource allocation.
- **Location**: [`docs/plans/implementation_plan.md`](docs/plans/implementation_plan.md)

### Testing Plans

- **Purpose**: Define how the system will be tested to ensure quality.
- **Contents**: Test cases, testing procedures, and expected outcomes.
- **Location**: [`docs/plans/testing_plan.md`](docs/plans/testing_plan.md)
> *Note*: Currently none, but planned to be created in the future. Can include both automated and manual testing.

### Development Logs

- **Purpose**: Store AI conversation logs and development notes for context and reference.
- **Contents**: 
  - *AI Conversations*: ChatGPT conversation logs that provide context for development decisions
  - *Development Notes*: Optional manual logs tracking progress, decisions, and changes
- **Location**:
`docs/logs/`<br>
`├── `[`ai_conversations/`](docs/logs/ai_conversations/)<br>
`└── `[`dev_notes/`](docs/logs/dev_notes/)

Possible future additions:
```
    ### Guides

    - **Purpose**: Provide guides for users and developers.
    - **Contents**:
    - *User Guide*, which provides instructions for users on how to use the system.
    - **Location**: `docs/guides/`
```

# Repository Structure

All documentation is organized within the `/docs/` directory of the repository.
