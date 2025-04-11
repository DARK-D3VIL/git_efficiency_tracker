# Employee & Reviewer Efficiency Tracker

A web-based dashboard to track developer and reviewer efficiency by analyzing GitHub activity. Built with **Ruby on Rails**, using **Devise for authentication** and **SQLite** for local data storage. Designed for organizations to assess performance based on PR activity and review engagement — without sharing GitHub tokens with end users.

---

## OBJECTIVE

To help organizations track engineering performance by:
    - Analyzing pull requests and code reviews
    - Measuring metrics like merge time, code quality, and review engagement
    - Suggesting improvements based on trends

---

## REQUIREMENTS

### Functional Requirements

#### User Auth System

    - Users sign up/login with email & password
    - GitHub token stored securely in backend (not shared with users)

#### Data Fetching

    - Uses a central GitHub token to access organization repositories
    - Pulls PR and review data across all repositories

#### Developer Tracking

    - Shows PR frequency, merge speed, code quality, and response to feedback

#### Reviewer Tracking

    - Shows review count, comment depth, response time, engagement, and helpfulness

#### Interactive Dashboard

    - Sorted lists of developers/reviewers
    - Search & filter functionality
    - Profile view for detailed metrics

#### Performance Suggestions

    - Recommends improvements based on developer/reviewer activity patterns

---

### 🔹 Non-Functional Requirements

| Component        | Stack                   |
|------------------|--------------------------|
| Backend          | Ruby on Rails            |
| Frontend         | Rails Views + Bootstrap  |
| Database         | SQLite (or PostgreSQL)   |
| Auth             | Devise                   |
| Token Security   | GitHub token + user IDs encrypted |

---

## 🔁 WORKFLOW

![alt text](<Editor _ Mermaid Chart-2025-04-10-113844.png>)

### 1️⃣ User Flow

[User Signup/Login]
     ↓
[Dashboard: Select Developer or Reviewer View]
     ↓
[List View: Can be Sorted by Performance Metrics]
     ↓
[Detail View: Developer or Reviewer Metrics]

### 2️⃣ GitHub Data Flow

    - Uses a single secure GitHub token
    - Fetches organization repositories:
  GET /orgs/:org/repos

- For each repository:
  - Fetch PRs:
    GET /repos/:owner/:repo/pulls?state=all
  - For each PR:
    - Get metadata (LoC, dates)
    - Commits
    - Review comments
    - Review approvals

- Compute and store:
  - Developer metrics
  - Reviewer metrics

---

## DATABASE STRUCTURE

![alt text](<Editor _ Mermaid Chart-2025-04-10-114418.png>)

### `users` – App Users

create_table :users do |t|
  t.string :email, null: false, unique: true
  t.string :password_digest
  t.string :encrypted_user_id
  t.timestamps
end

### `repositories`

create_table :repositories do |t|
  t.string :name
  t.string :full_name   # e.g., 'org/project-name'
  t.string :github_id
  t.timestamps
end

### `developers`

create_table :developers do |t|
  t.string :github_username
  t.string :github_id

  t.integer :total_prs, default: 0
  t.integer :total_commits, default: 0
  t.float   :avg_merge_time, default: 0.0
  t.float   :avg_feedback_response_time, default: 0.0
  t.float   :bug_fix_rate, default: 0.0
  t.float   :avg_code_quality_score, default: 0.0
  t.integer :total_lines_added, default: 0
  t.integer :total_lines_removed, default: 0
  t.integer :high_churn_prs, default: 0

  t.timestamps
end

### `reviewers`

create_table :reviewers do |t|
  t.string :github_username
  t.string :github_id

  t.integer :total_reviews, default: 0
  t.float   :avg_time_to_first_review, default: 0.0
  t.float   :avg_comments_per_pr, default: 0.0
  t.integer :total_repeated_comments, default: 0
  t.integer :total_unresolved_comments, default: 0
  t.float   :engagement_score, default: 0.0
  t.float   :helpfulness_score, default: 0.0

  t.timestamps
end

### `pull_requests`

create_table :pull_requests do |t|
  t.references :developer, foreign_key: true
  t.references :repository, foreign_key: true

  t.string :title
  t.string :github_pr_id
  t.datetime :opened_at
  t.datetime :merged_at

  t.integer :lines_added
  t.integer :lines_removed
  t.integer :review_comments_count
  t.boolean :is_bug_fix

  t.float :merge_time         # in hours
  t.float :response_time      # to feedback
  t.float :code_quality_score

  t.timestamps
end

### `reviews`

create_table :reviews do |t|
  t.references :reviewer, foreign_key: true
  t.references :pull_request, foreign_key: true

  t.integer :comments_count
  t.integer :repeated_comments
  t.integer :unresolved_comments
  t.integer :upvotes

  t.datetime :first_commented_at
  t.float :time_to_first_review

  t.timestamps
end

---

## 📅 4-Day Roadmap – GitHub-Based Employee & Reviewer Efficiency Tracker

This roadmap outlines the step-by-step development plan for building a GitHub-based employee efficiency tracker using Ruby on Rails, SQLite, and Devise.

---

### 🧭 Overview

- Track developers and reviewers by analyzing GitHub PR and review activity.
- Use a central GitHub token stored on the server (not shared with users).
- Developers are identified from PR creators; reviewers from PR review comments.
- PRs are classified as bug fixes if the linked GitHub issue has the label `bug`.

---

## 📅 Day 1: Project Initialization & Authentication

- Set up a Ruby on Rails project using SQLite.
- Install and configure Devise for user authentication.
- Users sign up/login with email and password only.
- Store GitHub token and organization name securely on the backend.
- Create base models: `User`, `Repository`, `Developer`, `Reviewer`, `PullRequest`, `Review`.
- Add `encrypted_user_id` to the `users` table (if needed for GitHub mapping).
- Set up `.env` file for GitHub credentials (local dev only).
- Confirm database migrations and Devise flow work as expected.

---

## 📅 Day 2: GitHub Integration & Data Sync

- Create a GitHub service class to interact with the GitHub API.
- Use a single, secure GitHub token from the backend to:
  - Fetch all repositories of the organization (`GET /orgs/:org/repos`)
  - Fetch PRs for each repository (`GET /repos/:owner/:repo/pulls?state=all`)
- For each PR:
  - Record metadata: title, opened/merged time, LoC added/removed.
  - Save PR to `pull_requests` table and associate with the `repository`.
  - Identify and store the PR creator as a `developer`.
- Fetch associated commits, review comments, and review approvals.
- Classify reviewers from users who leave review comments.
- Save all review data in the `reviews` table and link to `reviewers`.

---

## 📅 Day 3: Metrics Calculation & Logic Implementation

- Compute key developer metrics:
  - PR Frequency
  - Merge Speed (PR created to merged time)
  - Code Quality Score (based on churn, review comments, bug-fix ratio)
  - Response to Feedback (time from review comment to next commit)
- Compute key reviewer metrics:
  - Total Reviews
  - Avg. Time to First Review
  - Comments per PR
  - Engagement Score
  - Helpfulness Score (based on upvotes, if available)
- Identify bug-fix PRs:
  - Check if PR is linked to an issue with a `bug` label
  - Mark such PRs as bug fixes
- Store metrics in `developers` and `reviewers` tables for easy access.
- Build a GitHub sync controller to pull data on demand.
- Optionally prepare a background job for periodic syncs.

---

## 📅 Day 4: API Development & Dashboard Views

- Build API endpoints:
  - `GET /api/v1/developers` – list developers with sorting
  - `GET /api/v1/developers/:id` – show detailed developer metrics
  - `GET /api/v1/reviewers` – list reviewers with sorting
  - `GET /api/v1/reviewers/:id` – show detailed reviewer metrics
  - `POST /api/v1/sync/repositories` – trigger repository sync
  - `POST /api/v1/sync/data` – trigger PR + review data sync
  - `GET /api/v1/search?q=username` – search developers/reviewers
- Create UI with Bootstrap or Tailwind CSS:
  - Developer & Reviewer list views with filters
  - Individual profile views showing calculated metrics
- Secure dashboard routes using `before_action :authenticate_user!`
- Validate full system workflow: GitHub sync → metrics → display
- Finalize README, flowchart, ERD, and deployment notes

---

## 🧠 Critical Details

- PR creator → **Developer**
- Review comment author → **Reviewer**
- Bug-fix PR → PR linked to issue with `bug` label
- Users never see or control the GitHub token
- All GitHub interactions handled by the backend
- Handle GitHub pagination and rate limits carefully

---

## API STRUCTURE

| Method | Endpoint                       | Description                       |
|--------|--------------------------------|-----------------------------------|
| POST   | `/api/v1/signup`               | Sign up with email & password     |
| POST   | `/api/v1/login`                | Login & receive auth token        |
| GET    | `/api/v1/developers`           | List developers (sortable)        |
| GET    | `/api/v1/developers/:id`       | Developer profile view            |
| GET    | `/api/v1/reviewers`            | List reviewers (sortable)         |
| GET    | `/api/v1/reviewers/:id`        | Reviewer profile view             |
| POST   | `/api/v1/sync/repositories`    | Sync organization repositories    |
| POST   | `/api/v1/sync/data`            | Sync PRs and reviews for org      |
| GET    | `/api/v1/search?q=dev123`      | Search for developer/reviewer     |

---
