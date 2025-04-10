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
