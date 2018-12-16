---
layout: post
title: git 清空所有commit记录方法
---
**说明：例如将代码提交到git仓库，将一些敏感信息提交，所以需要删除提交记录以彻底清除提交信息，以得到一个干净的仓库且代码不变**


1. Checkout

    `git checkout --orphan latest_branch`

2. Add all the files

    `git add -A`

3. Commit the changes

    `git commit -am "commit message"`

4. Delete the branch
`
   ` git branch -D master

5. Rename the current branch to master

   ` git branch -m master`

6. Finally, force update your repository

   ` git push -f origin master`

