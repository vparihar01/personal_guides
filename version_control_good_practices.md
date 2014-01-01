# Version Control Best Practices
Today, version control should be part of every developer’s tool kit. Knowing the basic rules, however, makes it even more useful. Some of the best practices that help you get the most out of version control with Git.
In distributed version control, each user gets his or her own repository and working copy. After you commit, others have no access to your changes until you push your changes to the central repository. When you update, you do not get others' changes unless you have first pulled those changes into your repository. For others to see your changes, 4 things must happen:

* You commit
* You push
* They pull
* They update


## 1. Commit Related Changes

A commit should be a wrapper for related changes. Instead of committing many small changes bundled into one large chunk. This will reduce the chances of encountering complications such as merge conflicts, and will reduce the complexity of such conflicts when they do occur.For example, fixing two different bugs should produce two separate commits. Small commits make it easier for other developers to understand the changes and roll them back if something went wrong. With tools like the staging area and the ability to stage only parts of a file, Git makes it easy to create very granular commits.

## 2. Make each commit a logical unit

Each commit should have a single purpose and should completely implement that purpose. This makes it easier to locate the changes related to some particular feature or bug fix, to see them all in one place, to undo them, to determine the changes that are responsible for buggy behavior, etc.

## 3. Commit Often

Committing often keeps your commits small and, again, helps you commit only related changes. Moreover, it allows you to share your code more frequently with others. That way it’s easier for everyone to integrate changes regularly and avoid having merge conflicts. Having few large commits and sharing them rarely, in contrast, makes it hard to solve conflicts.

## 3. Don’t Commit Half-Done Work

You should only commit code when it’s completed. This doesn’t mean you have to complete a whole, large feature before committing. Quite the contrary: split the feature’s implementation into logical chunks and remember to commit early and often. But don’t commit just to have something in the repository before leaving the office at the end of the day. If you’re tempted to commit just because you need a clean working copy (to check out a branch, pull in changes, etc.) consider using Git’s "Stash" feature instead.

## 4. Share your changes more often

Once you have committed the changes for a complete, logical unit of work, you should share those changes with your colleagues as soon as possible (by doing git push). So long as your changes do not destabilize the system, do not hold the changes locally while you make unrelated changes. 

## 5. Don't leave commented-out code in there

Commented-out code pollutes the code base and makes no sense when you can easily check the history of a file.

## 4. Build and Test Before You Commit

Resist the temptation to commit something that you "think" is completed. Test it thoroughly to make sure it really is completed and has no side effects (as far as one can tell). While committing half-baked things in your local repository only requires you to forgive yourself, having your code tested is even more important when it comes to pushing / sharing your code with others.

## 5. Write Good Commit Messages

Begin your message with a short summary of your changes (up to 50 characters as a guideline). Separate it from the following body by including a blank line. The body of your message should provide detailed answers to the following questions: What was the motivation for the change? How does it differ from the previous implementation? Use the imperative, present tense ("change", not "changed" or "changes") to be consistent with generated messages from commands like git merge.

## 6. Version Control is not a Backup System

Having your files backed up on a remote server is a nice side effect of having a version control system. But you should not use your VCS like it was a backup system. When doing version control, you should pay attention to committing semantically (see "related changes") – you shouldn’t just cram in files.

## 7. Be consistent with your file/folder names



## 7. Use Branches

Branching is one of Git’s most powerful features – and this is not by accident: quick and easy branching was a central requirement from day one. Branches are the perfect tool to help you avoid mixing up different lines of development. You should use branches extensively in your development workflows: for new features, bug fixes, ideas...

## 8. Agree on a Workflow

Git lets you pick from a lot of different workflows: long-running branches, topic branches, merge or rebase, git-flow… Which one you choose depends on a couple of factors: your project, your overall development and deployment workflows and (maybe most importantly) on your and your teammates’ personal preferences. However you choose to work, just make sure to agree on a common workflow that everyone follows.
