---
date: 2024-03-29
tags:
  - github
  - 主页
publish: yes
---
# github action自动推送个人主页更新

采用[Host on GitHub Pages | Hugo (gohugo.io)](https://gohugo.io/hosting-and-deployment/hosting-on-github/)方案
{{< figure src="/attachment/Pasted%20image%2020240329165546.png" alt="Pasted image 20240329165546" width="500" >}}
推送到github后可见

{{< figure src="/attachment/Pasted%20image%2020240329190649.png" alt="Pasted image 20240329190649" width="800" >}}

类似的问题
[自动部署出现错误 · Issue #196 · YunYouJun/hexo-theme-yun (github.com)](https://github.com/YunYouJun/hexo-theme-yun/issues/196)


![Pasted image 20240329192546](/attachment/Pasted%20image%2020240329192546.png)

[Git - 子模块 (git-scm.com)](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

重新add一个submodule，比如paper主题，然后用git status可以查看到状态，明显有一个更改
{{< figure src="/attachment/Pasted%20image%2020240329194532.png" alt="Pasted image 20240329194532" width="500" >}}
这里记录了正确的路径，这次再部署应该没问题了

推送后得到结果
{{< figure src="/attachment/Pasted%20image%2020240329194917.png" alt="Pasted image 20240329194917" width="500" >}}
大坑，气死啦

>[! info] 总结
>关键是submodule的path要有且对


设置为了私密，更新一个post后，push发现失败

尝试回退版本并强制推送到远程
```shell
git reset --hard HEAD^
git push -f
```
无效，且content消失

重启开始site
有效，怀疑是更改了repo的权限导致的

成功的模样
{{< figure src="/attachment/Pasted%20image%2020240329210303.png" alt="Pasted image 20240329210303" width="500" >}}

<del>- [ ] 测试下repo为私密时的情况⏳ 2024-03-30 </del>
- [x] 修改主页样式（hugo.toml文件） ✅ 2024-03-30
- [x] 书写写作记录⏳ 2024-03-30  ✅ 2024-03-30

---
refs:
https://ranch007.github.io/p/2026-07-04-obsidian-github-hugo-博客发布工作流/
https://blog.csdn.net/Florae006/article/details/161120676



