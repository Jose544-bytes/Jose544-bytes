- 👋 Hi, I’m @Jose544-bytes
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Jose544-bytes/Jose544-bytes is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
name: run-checks

permissions: read-all

on:
  merge_group:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true

jobs:
  main-checks:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        # https://github.com/actions/virtual-environments#available-environments
        os: [ubuntu-latest, macos-latest, windows-latest]
        # https://github.com/nodejs/Release#release-schedule
        node: [18, 20]
    steps:
      - name: Checkout
        uses: actions/checkout@44c2b7a8a4ea60a981eaca3cf939b5f4305c123b # v4.0.0
        with:
          fetch-depth: 2

      - name: Set up Node.js
        uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8 # v4.0.2
        with:
          node-version: ${{ matrix.node }}

      - name: Install dependencies
        run: |
          npm ci

      - name: Build
        run: |
          npm run build

      - name: Run code checks
        run: |
          npm run lint

      - name: Run tests
        uses: nick-invision/retry@7152eba30c6575329ac0576536151aca5a72780e # v3.0.0
        with:
          max_attempts: 3
          command: npm run test
          timeout_minutes: 10
  headful-checks:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest]
        node: [20]
    steps:
      - name: Checkout
        uses: actions/checkout@44c2b7a8a4ea60a981eaca3cf939b5f4305c123b # v4.0.0
        with:
          fetch-depth: 2

      - name: Set up Node.js
        uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8 # v4.0.2
        with:
          node-version: ${{ matrix.node }}

      - name: Install dependencies
        run: |
          sudo apt-get install xvfb
          npm ci

      - name: Build
        run: |
          npm run build

      - name: Run tests in headful
        uses: nick-invision/retry@7152eba30c6575329ac0576536151aca5a72780e # v3.0.0
        with:
          max_attempts: 3
          command: xvfb-run --auto-servernum npm run test:headful
          timeout_minutes: 10
