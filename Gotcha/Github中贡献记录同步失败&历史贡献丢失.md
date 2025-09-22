#Github
- **Problem**: Github中最近的贡献记录丢失问题，导致在贡献图标中看不到自己的历史提交记录。
- **Solution**: Fixed advice ref. : [Troubleshooting missing contributions](https://docs.github.com/en/account-and-profile/how-tos/setting-up-and-managing-your-github-profile/managing-contribution-settings-on-your-profile/troubleshooting-missing-contributions).
	- 恢复将来的贡献记录：
		[Your local Git commit email isn't connected to your account](https://docs.github.com/en/account-and-profile/how-tos/setting-up-and-managing-your-github-profile/managing-contribution-settings-on-your-profile/troubleshooting-missing-contributions#your-local-git-commit-email-isnt-connected-to-your-account)
		GitHub仅在贡献图表中记录 GitHub上的帐户的电子邮件地址的提交记录。
  ```sh
	git config user.email
	git config --global user.email "YourGitHubEmailAddress"
	git config user.email
	```
	-   恢复历史的贡献记录：
		使用官方工具`{shell icon} git-filter-repo `修改历史提交日志，重新推送到Github中即可。
```sh
	# 1. 安装 git-filter-repo 重写历史提交的邮箱
	pip install git-filter-repo
	
	# 2. 确认log中历史提交邮箱信息
	git log --format="%an <%ae>" 
	
	# 3.1 使用 git filter-repo 修改历史提交的邮箱
	git filter-repo --email-callback ' return email if email != b"OldYourGitHubEmailAddress" else b"NewYourGitHubEmailAddress" ' 
	# 3.2 如果有多个历史邮箱需要替换，执行下述脚本
	git filter-repo --email-callback ' if email in [b"OldYourGitHubEmailAddress1", b"OldYourGitHubEmailAddress2"]: return b"NewYourGitHubEmailAddress" return email '
	
	# 4. 确认log中历史提交邮箱修改成功
	git log --format="%an <%ae>" 
	# 5. 推送到 GitHub
	git push --force
	```
- **Note**: Watch out for [gotcha/pitfall] in future (e.g., check version compatibility)
