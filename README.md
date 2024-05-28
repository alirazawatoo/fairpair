# FairPair

FairPair is a Rails application for rotating developers through sprint pairings. A team lead creates projects, assigns developers, chooses the number of sprints, and generates a round-robin schedule designed to spread collaboration across the team.

The project is a compact example of a server-rendered Rails product with role-aware authorization, domain services, reusable view components, Hotwire interactions, and automated tests.

## Product workflow

```text
Lead signs in
    |
    v
Creates a project -----> assigns developers
    |                           |
    +---------------------------+
                |
                v
        configures sprint count
                |
                v
   round-robin pairing service
                |
                v
      sprint-by-sprint schedule
```

### Lead capabilities

- Create, edit, and remove projects
- Create developer accounts in a reporting hierarchy
- Add or remove developers from a project
- Configure the project’s sprint count
- Generate pairings across the available developers

### Developer model

Developers belong to a lead and can be assigned to multiple projects through `UserProject`. Project and user policies use Pundit to constrain lead-only workflows.

## How pairing works

`ShuffleSprintDevelopers` is the application’s central domain service. It:

1. Resolves the current user and project.
2. Reconciles the requested number of persisted sprints.
3. Builds unique developer pairs with a round-robin schedule.
4. Repeats pairings when the sprint count is longer than the unique schedule.

Keeping this logic outside controllers makes the scheduling rule easier to inspect and test.

## Architecture

```text
app/
├── components/    # Reusable ViewComponent UI units
├── controllers/   # Project, developer, sprint, and membership flows
├── models/        # Users, projects, sprints, and project membership
├── policies/      # Pundit authorization rules
├── services/      # Round-robin sprint scheduling
└── views/         # Rails and Turbo templates

spec/              # RSpec model, request, controller, and service-adjacent coverage
test/              # Rails controller and component coverage
```

## Technology

- Ruby 2.7.1 and Rails 7
- PostgreSQL
- Devise authentication
- Pundit authorization
- Hotwire, Stimulus, and Turbo
- ViewComponent
- Tailwind CSS and Flowbite
- RSpec, FactoryBot, and Rails tests

## Local setup

### Prerequisites

- Ruby 2.7.1
- PostgreSQL
- Bundler

```bash
git clone https://github.com/alirazawatoo/fairpair.git
cd fairpair
bundle install
bin/rails db:prepare
bin/dev
```

Open `http://localhost:3000`. Seed data is available through `bin/rails db:seed` if you want a populated development database.

## Tests

The repository contains both RSpec and Rails test suites:

```bash
bundle exec rspec
bin/rails test
```

Coverage includes models, controllers, request flows, policies exercised through controllers, and individual UI components. The commands above are the intended checks; results are not claimed here unless reproduced in the target environment.

## Design choices

- **Service object for scheduling:** pairing rules stay separate from HTTP and rendering concerns.
- **Explicit project membership:** the join model supports many-to-many project assignments.
- **Policy layer:** lead-only operations are expressed through Pundit rather than scattered role checks.
- **Componentized UI:** repeated team and sprint views live in ViewComponent classes and templates.
- **Progressive enhancement:** Turbo streams update project and developer workflows without requiring a separate SPA.

## Current boundaries

- Pairing is based on developer membership, not skills, availability, or prior sprint outcomes.
- The application assumes a single lead hierarchy rather than organization-level tenancy.
- There is no public deployment configuration or hosted demo in the repository.
- Some behavior is covered by both Rails tests and RSpec; consolidating conventions would reduce maintenance overhead.

## Next improvements

1. Add schedule history so leads can audit and compare rotations.
2. Enforce database-level uniqueness for project memberships.
3. Add acceptance coverage for the full project-to-schedule workflow.
4. Introduce organization boundaries before supporting multiple companies.
