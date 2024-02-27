# 理想架構使用法

# Git
  ## Git 內容定義
  本文主要有三個線呈:branch.develop.master對應 本機.測試機.正式機。
  測試機網址會設定為 test.xxx
  正式機網址會設定為 xxx
  
  ## Git CI/CD
  必須架構在 GitLab Runner上，如果在 *特定主機* 會需要權限。
  透過 '.yml' 來控制，以下為限制只有在 **master** 分支時，才構建和部署 Docker container的範例。
  ```yml
  stages:          # List of stages for jobs, and their order of execution
    - build
    - deploy
  
  build:
    stage: build
    script:
      - echo "Building the Docker image..."
      - docker build -t my-app:$CI_COMMIT_REF_NAME .
    only:
      - master
  
  deploy:
    stage: deploy
    script:
      - echo "Deploying the Docker image..."
      - docker push my-app:$CI_COMMIT_REF_NAME
    only:
      - master
  ```

  ## 部屬在特定主機之上與測試

# Docker為主的部屬方式
  ## 好處
  上版、錯誤不影響他人，退版也方便。

  ## 達成方法
  使用 **K8s** 配合 **nginx** 進行多線任務分配，並且以一個 **Docker** 包一個 **API** 為原則。
  也就是說每一個 **API** 資料夾中都會包自己的 **config** 。

  ### K8s介紹優文 < https://ithelp.ithome.com.tw/articles/10192401>  
  
# K8s個人的小廢物介紹
