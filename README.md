# alpro-tugas1
name: Branching Strategy Workflow

on:
  push:
    branches:
      - main
      - hotfixes/*
      - release/*
      - feature/*

  pull_request:
    branches:
      - main
      - hotfixes/*
      - release/*
      - feature/*

jobs:
  # Job untuk membuat branch baru (feature, hotfix, release) secara otomatis saat push atau pull request
  create-branch:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Git
        run: |
          git config --global user.name "github-actions"
          git config --global user.email "github-actions@github.com"

      - name: Checkout and create branch
        run: |
          BRANCH_NAME=$(echo $GITHUB_REF | sed 's|refs/heads/||')
          echo "Creating branch: $BRANCH_NAME"
          git checkout -b $BRANCH_NAME
          
      - name: Push new branch to remote
        run: |
          git push origin $BRANCH_NAME

  # Job untuk build dan deploy, tergantung pada jenis branch
  build-and-deploy:
    runs-on: ubuntu-latest
    needs: create-branch
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Build project
        run: |
          echo "Building project for branch $GITHUB_REF"

      - name: Deploy to production (main)
        if: github.ref == 'refs/heads/main'
        run: |
          echo "Deploying to production environment"

      - name: Deploy to staging (release)
        if: github.ref == 'refs/heads/release/*'
        run: |
          echo "Deploying to staging environment"

      - name: Deploy to test (feature)
        if: github.ref == 'refs/heads/feature/*'
        run: |
          echo "Deploying to test environment"

      - name: Deploy to fix environment (hotfix)
        if: github.ref == 'refs/heads/hotfixes/*'
        run: |
          echo "Deploying to hotfix test environment"
