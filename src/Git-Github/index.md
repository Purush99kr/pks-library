# Git and GitHub learning guide

<img src="image/github.png" alt="Description" width="1200" height="400">

## What is [GitHub](https://github.com/)?

GitHub is a cloud-based platform where developers store, manage, track, and collaborate on software code. Think of it as a **_social network for programmers_** combined with an ultra-secure cloud backup for project files.

It is built on top of **_Git_**, an open-source **_version control system_** that records every history change made to a file so you can easily revert to older versions if something breaks.

## Who Uses GitHub?

While designed primarily for technical roles, GitHub Docs states that it is widely adopted by various professionals:

- **_Software Engineers & Developers :_** Individual creators and global engineering teams. Track and store the project files.
- **_Students & Beginners :_** Those learning to code and building their first portfolios. Store learning notes globally.
- **_Data Scientists & Researchers :_** Academics tracking experimental scripts and datasets.
- **_Project Managers :_** Teams organizing development sprints and assigning tasks.
- **_Businesses & Enterprises :_** Companies protecting private codebases (including 90% of Fortune 100 companies).

## Access to the GitHub Repositories?

In order to use GitHub, you need the Git Config, a GitHub profile and a repository over the GitHub dashboard.

- To create GitHub profile, visit the [GitHub official dashboard](https://github.com/) and create a profile.
- Then after create a new repository and configure Git to store, track and manage your files.

## What is [Git](https://git-scm.com/)?

Git is a free and **_open-source_** distributed **_version control system_** created by Linus Torvalds in 2005. It tracks changes in files over time, allowing multiple people to work on the same project safely, review history, and undo mistakes without overwriting each other's work.

## Git Config in the system having multiple GitHub accounts

1.  Create SSH key for GitHub Account 1

    ```text
    ssh-keygen -t ed25519 -C "YOUR_ACCOUNT_1_EMAIL"
    ```

    When asked : Enter file in which to save the key : then provide file below

    ```text
    ~/.ssh/id_ed25519_"unique-ID"
    ```

    For passphrase, you can press Enter twice if you don't want one.

    This creates:

    ```text
    ~/.ssh/id_ed25519_unique-ID
    ~/.ssh/id_ed25519_unique-ID.pub
    ```

    If don't find **_.ssh_** folder : Run the below command below and process same as above (Don't use **_~/.ssh/..._** here; use the full Windows path.)

    ```text
    mkdir C:\Users\user\.ssh
    ```

2.  Similarly create SSH Key for another accounts
3.  Add SSH Keys to the specific GitHub accounts.

    Copy each Account's public key:

    ```text
    cat ~/.ssh/id_ed25519_unique-ID.pub | clip
    ```

    Then

    ```text
    GitHub → Settings → SSH and GPG keys → New SSH key
    ```

    Paste the key → Add SSH key.

    Do same for all accounts.

4.  Create SSH configuration

    ```text
    nano ~/.ssh/config
    ```

    If **_nano_** isn't available, create/open this file:

    ```text
    C:\Users\YOUR_USERNAME\.ssh\config
    ```

    Create the config file

    ```text
    - cd C:\Users\user\.ssh
    - New-Item -Path . -Name "config" -ItemType File
    ```

    Open file in editor and paste this into it

    ```text
    Host github-unique-ID
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_unique-ID
    IdentitiesOnly yes

    Host github-unique-ID
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_unique-ID
    IdentitiesOnly yes
    ```

5.  Test/Verify accounts:

    Run

    ```text
    ssh -T git@github-purush
    ```

    Output

    ```text
    Hi 'Yourname'! You've successfully authenticated...
    ```

## Connect Git with GitHub, push project code over GitHub repo

### Required before setup

- [Create a GitHub Repo](https://github.com/)
- [Git install](https://git-scm.com/)
- [Code Editor install](https://code.visualstudio.com/)

Create a GitHub repo and copy the repo url.

```text
copy only as  : "your-username/your-reponame"
```

Create a project in any tech and open project directory in the code editor terminal. And execute the commands below :

```text
- git status
- git init
- git config --local user.name "your-username"
- git config --local user.email "your-email"
- git add .
- git commit -m "Your commit message"
- git branch -M main
- git remote add origin git@github-<'SSH host alias'>:"your-username/your-reponame"
- git push -u origin main
```

## Delete **_.git_** folder from the project root : Run the command inside your project folder.

Windows PowerShell

```text
Remove-Item -Recurse -Force .git
```

Windows CMD

```text
rmdir /s /q .git
```

Windows Git Bash / macOS Terminal / Linux Terminal

```text
rm -rf .git
```
