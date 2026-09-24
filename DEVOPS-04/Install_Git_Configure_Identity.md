Task 2 – Install Git & Configure Identity
Step 1: Update Ubuntu Packages
sudo apt update

This updates the package list from Ubuntu repositories.

Step 2: Install Git
sudo apt install git -y
Screenshot 1

Capture the terminal after Git installation completes successfully.

Example:

Setting up git ...
Processing triggers ...
Step 3: Verify Git Installation

Run:

git --version

Example Output:

git version 2.43.0
Screenshot 2

Capture the output of:

git --version
Step 4: Configure Git Username

Replace with your actual name.

git config --global user.name "Sathvik Jakka"

Verify:

git config --global user.name

Expected Output:

Sathvik Jakka
Step 5: Configure Git Email

Replace with your email address.

git config --global user.email "your_email@example.com"

Verify:

git config --global user.email

Expected Output:

your_email@example.com
Step 6: View All Git Configuration

Run:

git config --list

Example Output:

user.name=Sathvik Jakka
user.email=your_email@example.com
core.editor=vim
init.defaultBranch=main
Screenshot 3

Capture the output of:

git config --list
Understanding Each Configuration Entry
user.name
user.name=Sathvik Jakka
Identifies the author of commits.
Appears in Git history.
user.email
user.email=your_email@example.com
Associates commits with your email.
Used by GitHub to link commits to your account.
init.defaultBranch
init.defaultBranch=main
Sets the default branch name when creating new repositories.
core.editor
core.editor=vim
Default text editor used by Git.
Commands Used
sudo apt update

sudo apt install git -y

git --version

git config --global user.name "Sathvik Jakka"

git config --global user.email "your_email@example.com"

git config --list