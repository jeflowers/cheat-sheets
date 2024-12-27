# Git Practice Exercise #

## **Introduction**
This step-by-step exercise will reinforce Git concepts using the example of a project called "Tracking App." You will practice initializing a repository, staging files, making commits, and reviewing the commit history. By following these steps, you’ll gain confidence in using Git effectively.

---

## **Steps to Practice Git**

### **1. Initialize a Git Repository**
1. Create a new folder for your project:
    ```bash
    mkdir tracking_app && cd tracking_app
    ```
2. Initialize a Git repository inside this folder:
    ```bash
    git init
    ```
    > This creates an empty Git repository with a `.git` directory.

---

### **2. Create Initial Files**
1. Add two new files:
    ```bash
    touch app_overview.txt data_sources.txt
    ```
2. Use `git status` to check the current state of your repository:
    ```bash
    git status
    ```
    > Both files will appear as "untracked."

---

### **3. Stage and Commit Files**
1. Add both files to the staging area:
    ```bash
    git add app_overview.txt data_sources.txt
    ```
2. Commit the staged files with a descriptive message:
    ```bash
    git commit -m "Initial commit: Add app overview and data sources files"
    ```

---

### **4. Update Files**
1. Open `app_overview.txt` and add the following content:
    ```plaintext
    This app tracks user activities and data metrics.
    ```
2. Open `data_sources.txt` and add:
    ```plaintext
    - User logs
    - System metrics
    ```

---

### **5. Make Separate Commits for Each File**
1. Stage and commit changes to `app_overview.txt`:
    ```bash
    git add app_overview.txt
    git commit -m "Update app overview with tracking details"
    ```
2. Stage and commit changes to `data_sources.txt`:
    ```bash
    git add data_sources.txt
    git commit -m "Add initial data sources list"
    ```

---

### **6. Modify and Commit Multiple Files Together**
1. Update `data_sources.txt` to include:
    ```plaintext
    - External APIs
    ```
2. Update `app_overview.txt` to mention:
    ```plaintext
    The app supports real-time tracking and historical data analysis.
    ```
3. Stage and commit both changes with a single message:
    ```bash
    git add .
    git commit -m "Update app overview and data sources"
    ```

---

### **7. View Commit History**
1. Use `git log` to review all commits made so far:
    ```bash
    git log
    ```
2. For a concise view of the commit history:
    ```bash
    git log --oneline
    ```

---

## **Key Takeaways**
- **Initialize**: Always start by creating and initializing a repository.
- **Stage and Commit**: Use `git add` to stage changes and `git commit` to save them.
- **Separate Commits**: Break changes into logical, atomic commits for clarity.
- **Review History**: Regularly check your commit history to ensure clarity and accuracy.
  
---

This exercise cultivates a profound comprehension of Git workflows, enhancing your mastery of version control with each step taken.
