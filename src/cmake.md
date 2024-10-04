# cmake 学习

## 参考资料
1. [cmake官方文档](https://cmake.org/cmake/help/latest/)
2. [effective_modern_cmake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1)
3. [modern cmake](https://cliutils.gitlab.io/modern-cmake/README.html)
4. [cmake中文参考](https://juejin.cn/post/6844903557183832078)
5. [视频1](https://www.bilibili.com/video/BV1fa411r7zp/?spm_id_from=333.999.0.0&vd_source=4703b3762c3876a731aa398192561d8f)
6. [视频2](https://www.bilibili.com/video/BV16P4y1g7MH/?spm_id_from=333.999.0.0&vd_source=4703b3762c3876a731aa398192561d8f)
7. [视频3](https://space.bilibili.com/263032155/channel/collectiondetail?sid=53025)


## 指定使用 clang 进行编译链接
```cmake
target_compile_options(a.out PRIVATE -stdlib=libc++)
target_link_options(a.out PRIVATE -stdlib=libc++)
```