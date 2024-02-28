# 目前基準架構設計法

# Git
  1. Git 內容定義
  本文主要有三個線呈:branch.develop.master對應 本機.測試機.正式機。     
  測試機網址會設定為 test.xxx  
  正式機網址會設定為 xxx  
  
  2. Git CI/CD
  必須架構在 GitLab Runner上，如果在 *特定主機* 會需要權限。不過在付費版據說可以線上設定，~~可能要研究一下~~         
  透過 '.gitlab-ci.yml' 來控制，以下為限制只有在 **master** 分支時，才構建和部署 Docker container的範例。  
  ```yml
  stages:          # List of stages for jobs, and their order of execution
    - build
    - test
    - deploy
  
  build:
    stage: build
    script:
      - echo "Building the Docker image..."
      - docker build -t my-app:$CI_COMMIT_REF_NAME .
    only:
      - master

  unit-test-job:   # This job runs in the test stage.
    stage: test    # It only starts when the job in the build stage completes successfully.
    before_script:
        - pip install -r requirements.txt
    script:
        - python -m pytest --junitxml=report.xml  # use PyTest
    artifacts:
        when: always
        reports:
            junit: report.xml
        expire_in: 2 weeks
    except:
      - branches

  test_job:       # 另一種寫單元測試的方法
  stage: test
  script:
    - echo "Running tests"
    - ./run_tests.sh
  only:
    - branches
  
  deploy:
    stage: deploy
    script:
      - echo "Deploying the Docker image..."
      - docker push my-app:$CI_COMMIT_REF_NAME
    only:
      - master
  ```

  *相關測試資料文章* < https://docs.gitlab.com/ee/ci/testing/ >

  3. develop.master部屬在特定主機之上與測試
     在 '.gitlab-ci.yml' 中定義環境。為了資料安全可用 GitLab 的 Secret Variables加密後儲存，不過這部分個人習慣呼叫Node.js/Express.js傳輸加密。     
     [UI設定環境的方式](https://ithelp.ithome.com.tw/articles/10251547 "link")      [在yml設定環境變數的方式](https://ithelp.ithome.com.tw/articles/10251547 "link")
 ``` yaml
 stages:
  - test
  - deploy

test_job:
  stage: test
  script:
    - echo "Running tests..."
  only:
    - branches

deploy_to_staging:
  stage: deploy
  script:
    - echo "Deploying to the test environment..."
  environment:
    name: staging
  only:
    - develop

deploy_to_production:
  stage: deploy
  script:
    - echo "Deploying to the production environment..."
  environment:
    name: production
  only:
    - master
 ```

# Docker為主的部屬方式
  1. 好處
  上版、錯誤不影響他人，退版也方便。  

  2. 達成方法
  使用 **K8s** 配合 **nginx** 進行多線任務分配，並且以一個 **Docker** 或 **Pod** 包一個 **API** 為原則。
  也就是說每一個 **API** 資料夾中都會包自己的 **config** 。

  3. K8s介紹優文 <https://ithelp.ithome.com.tw/articles/10192401>    <https://github.com/zxcvbnius/k8s-30-day-sharing/tree/master>  
  
# K8s使用以及可能遇到的狀況(以下可以先不要看，還在整理)
  1. 介紹
     透過master分派任務給node工作，並且達成熱更新(更新不用重啟系統)、API錯誤會更新失敗的狀況。
     
  2. 安裝
     Kubeadm或者[github](https://github.com/kubernetes/kubernetes"link")的檔案。~~拜託不要只能裝github，有很多權限要補~~
     在此用祈禱不要卡在作業系統的proxy還有containerd 的proxy上...聽說封閉內網會有很多意想不到的事件。                           
     
     * Kubeadm
     ..創建master節點kubeadm init後，將node節點用kubeadm join到Master的IP和端口。
        ** 硬體限制      
     ![image](https://github.com/candy229909/GitSetSpecifyServer/assets/52559100/1dfa7a52-672b-4979-b821-e29e8091b034)

      ** 指令
      ``` terminal
      $ kubeadm init
      $ kubeadm join
       ```

      ![image](https://github.com/candy229909/GitSetSpecifyServer/assets/52559100/be527f03-8d6e-4643-a6d0-025055b200a4)
     ![image](https://github.com/candy229909/GitSetSpecifyServer/assets/52559100/a426ea2e-7770-4a0a-89ee-b0ff8489dabd)


     

  4. 自動擴展
    < https://ithelp.ithome.com.tw/articles/10193856>
  6. 單一master
  7. 多個master
