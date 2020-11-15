- 查看Tag

  - 查看所有tag

    ```shell
    git tag
    ```

  - 查看v1.x的tag

    ```shell
    git tag -l v1.*
    ```

- 创建一个tag

  ```shell
  git tag v1.0
  ```

- 删除一个tag

  ```shell
  git tag -d v1.0
  ```

- 将tag推送到远程仓库

  - 推送单个tag
  
    ```shell
  git push <name> v1.0
    ```
  
  - 将所有tag推送到远程仓库
  
    ```shell
    git push <name> --tags
    ```
  
  这里的`<name>`为远程仓库名，一般为`origin`

