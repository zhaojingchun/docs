## SSH免密与Token登录配置



[参考](https://cloud.tencent.com/developer/article/1861466)

大致意思是，密码验证于2021年8月13日不再支持，也就是今天intellij不能再用密码方式去提交代码。请用使用 **personal access token** 替代。



### create a new repository on the command line

注意要用ssh链接

 ```linux
 echo "# docs" >> README.md
 git init
 git add README.md
 git commit -m "first commit"
 git branch -M main
 git remote add origin git@github.com:xxx/docs.git
 git push -u origin main
 ```

### …or push an existing repository from the command line

```linux
git remote add origin git@github.com:xxx/docs.git
git branch -M main
git push -u origin main
```

