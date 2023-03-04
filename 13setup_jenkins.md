copy password di 
```shell
docker logs jenkins
```
![](.13setup_jenkins_images/bb41e30a.png)

paste ke website
![](.13setup_jenkins_images/.png)

select plugin to install
![](.13setup_jenkins_images/c8b09a62.png)

pilih plugin yang diperlukan
![](.13setup_jenkins_images/24ac7c82.png)

tunggu sampai proses instalasi selesai
![](.13setup_jenkins_images/539e10ab.png)

masukan form
![](.13setup_jenkins_images/74f4a57f.png)

masukan domain
![](.13setup_jenkins_images/a9a6bd40.png)

# setup credentials di jenkins

pada dashboard jenkins, pergi ke manage jenkins > manage credentials > global >
add credentials
![](.13setup_jenkins_images/c4db0bf7.png)

masukan private key
![](.13setup_jenkins_images/8246954d.png)



# setup credentials di jenkins
pada website github, pergi ke setting > SSH and GPG keys > new SSH key, masukan
public key
![](.13setup_jenkins_images/af794888.png)

SSH key berhasil ditambahkan
![](.13setup_jenkins_images/3b4079f2.png)


# setup agar bisa terkoneksi dengan github di koneksi pertama
pilih manage jenkins > configure global security 
![](.13setup_jenkins_images/4e86637a.png)

scroll ke bagian paling bawah, accept first connection
![](.13setup_jenkins_images/99d90b8a.png)

# setup notifikasi dengan slack
install plugin slack notification, dashboard > manage jenkins, manage plugins 
![](.13setup_jenkins_images/15f1f5aa.png)

pada available plugins, cari plugin slack notification, pilih download now and isntall after restart 
![](.13setup_jenkins_images/f13f0e7d.png)

tunggu sampai proses instalasi berhasil
![](.13setup_jenkins_images/37572a49.png)

buka halaman https://my.slack.com/services/new/jenkins-ci, lalu pilih workspace 


pilih channel
![](.13setup_jenkins_images/43cb76ee.png)

setelah itu scroll kebawah, kita akan menemukan team subdomain, dan integration token credential ID 
![](.13setup_jenkins_images/db5406ea.png)

copy integration token credential ID, lalu tambahkan di credential jenkins, pada menu kind, pilih secret text
![](.13setup_jenkins_images/1e20ed38.png)

setelah itu, pada dashboard jenkins > manage jenkins > configure system, cari menu slack, masukan form yang diperlukan, untuk workspace masukan team subdomain tadi, 
disarankan untuk test connection terlebih dahulu sebelum save 
![](.13setup_jenkins_images/921486c5.png)

# setup github repository
buat private github repository
![](.13setup_jenkins_images/8224d28c.png)

pada folder frontend, tambahkan branch remote
```shell
git remote add origin git@github.com:Reiya24/fe-dumbmerch.git
```
![](.13setup_jenkins_images/4bbf9ac6.png)


push semua branch ke github
```shell
git push origin --all
```
![](.13setup_jenkins_images/058626b3.png)

push berhasil
![](.13setup_jenkins_images/13cc4c50.png)

# setup github webhook
pada project github, pilih settings, webhook, add webhook
![](.13setup_jenkins_images/58fe368d.png)

masukan URL jenkins
![](.13setup_jenkins_images/7ba235ee.png)

integrasi berhasil
![](.13setup_jenkins_images/17af0028.png)

# membuat pipeline

klik create job, pilih  pipeline job
![](.13setup_jenkins_images/dd2d08ed.png)

ceklis github hook trigger for scm polling
![](.13setup_jenkins_images/eefff909.png)

definition, pilih git, masukan repository, ssh ke
y, dsb
![](.13setup_jenkins_images/b5a6e3a2.png)

matikan lightweight checkout

pada direktori frontend, buat Jenkinsfile
```shell
pipeline {
    agent any

    environment{
        def branch = "production"
        def nama_repository = "origin"
        def directory = "~/fe-dumbmerch"
        def credential = 'id_rsa'
        def server = 'reiya24@10.116.106.150'
        def docker_image = 'reiya24/dumbmerch-frontend-production'
        def nama_container = 'frontend-production'
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 40, unit: 'MINUTES')
    }

    stages {

        stage('kirim notifikasi ke slack') {
            steps {
                slackSend(message: "mulai job baru : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }

        stage('pull repository dari github ') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    git pull ${nama_repository} ${branch}
                    exit
                    EOF"""
                }
            }
        }

        stage('hapus container') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    docker container stop ${nama_container}
                    docker container rm ${nama_container}
                    exit
                    EOF"""
                }
            }
        }

        stage('build image frontend') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker build -t ${docker_image}:latest .
                    exit
                    EOF"""
                }
            }
        }

        stage('jalankan docker compose') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose up -d
                    exit
                    EOF
                    """
                }
            }
        }
        
        stage('push image ke dockerhub') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker image push ${docker_image}:latest
                    exit
                    EOF"""
                }
            }
        }
    }

    post {

        aborted {
            slackSend(message: "build digagalkan secara manual : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(message: "build failed : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        success {
            slackSend(message: "build success : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        
    }
}
```

push ke github repository
![](.13setup_jenkins_images/dcadd45a.png)

jenkins akan membuild secara otomtatis
![](.13setup_jenkins_images/88ee37a7.png)

notifikasi berhasil
![](.13setup_jenkins_images/52f30ce1.png)

## lakukan hal yang sama untuk backend
![](.13setup_jenkins_images/2075db3f.png)

# multibranch
install plugin multibranch scan webhook trigger
![](.13setup_jenkins_images/8c515b42.png)

pilih new item

masukan nama branch, pilih multibranch pipeline
![](.13setup_jenkins_images/d8201718.png)

setup branch source
![](.13setup_jenkins_images/5cccd3b7.png)

ceklis scan by webhook
![](.13setup_jenkins_images/2ed3861b.png)

jenkins akan mengscan branch yang memiliki Jenkinsfile
![](.13setup_jenkins_images/64ddd34f.png)

```shell
pipeline {
    agent any

    environment{
        def branch = "production"
        def nama_repository = "origin"
        def directory = "~/fe-dumbmerch"
        def credential = 'id_rsa'
        def server = 'reiya24@10.116.106.150'
        def docker_image = 'reiya24/dumbmerch-frontend-production'
        def nama_container = 'frontend-production'
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 40, unit: 'MINUTES')
    }

    stages {

        stage('kirim notifikasi ke slack') {
            steps {
                slackSend(message: "mulai job baru : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
            }
        }

        stage('pull repository dari github ') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    git checkout ${branch}
                    git pull ${nama_repository} ${branch}
                    exit
                    EOF"""
                }
            }
        }

        stage('hapus container') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    docker container stop ${nama_container}
                    docker container rm ${nama_container}
                    exit
                    EOF"""
                }
            }
        }

        stage('build image frontend') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker build -t ${docker_image}:latest .
                    exit
                    EOF"""
                }
            }
        }

        stage('jalankan docker compose') {
            steps {
                sshagent([credential]){
                    sh"""ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose up -d
                    exit
                    EOF
                    """
                }
            }
        }
        
        stage('push image ke dockerhub') {
            steps {
                sshagent([credential]){
                    sh"""
                    ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker image push ${docker_image}:latest
                    exit
                    EOF"""
                }
            }
        }
    }

    post {

        aborted {
            slackSend(message: "build digagalkan secara manual : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        failure {
            slackSend(message: "build failed : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }

        success {
            slackSend(message: "build success : ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        
    }
}
```
![](.13setup_jenkins_images/ab929f94.png)

## setup github webhook
pada repository github > pilih setting > webhook, masukan
```shell
https://jenkins.reiya.my.id/multibranch-webhook-trigger/invoke?token=backend
```
![](.13setup_jenkins_images/06989371.png)

push ke github
![](.13setup_jenkins_images/47f377b7.png)

trigger berhasil
![](.13setup_jenkins_images/89990175.png)

![](.13setup_jenkins_images/2ef3d455.png)

![](.13setup_jenkins_images/a33fe039.png)