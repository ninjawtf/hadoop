pipeline {
    agent any

    tools {
        maven 'maven-3.8.6'
        jdk 'jdk-8'
    }
    stages {
        stage('Prerequisites') {
            steps {
                sh "sudo apt-get -y install build-essential autoconf automake libtool cmake zlib1g-dev pkg-config libssl-dev gcc-multilib g++-multilib lib32z1-dev openssl"
                sh ''' 
                    wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
                    tar -xvf protobuf-2.5.0.tar.gz
                    cd protobuf-2.5.0
                    ./configure --prefix=/usr
                    make
                    sudo make install
                    protoc --version
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -Pdist,native -DskipTests -Dtar"
            }
        }

        stage('Deploy'){
            steps {
                sh '''
                    if [ -f /opt/hadoop/sbin/stop-dfs.sh ]; then
                        /opt/hadoop/sbin/stop-dfs.sh
                    fi
                '''
                sh "rm -rf /opt/hadoop/*"
                sh "tar -C /opt/hadoop -xvf hadoop-dist/target/hadoop-3.1.*.tar.gz"
                sh "mv /opt/hadoop/hadoop-3.1.*/* /opt/hadoop/"
                
                sh "export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_231"
                sh "cp -f ./configs/single-node/* /opt/hadoop/etc/hadoop"
                sh "echo \"JAVA_HOME=/usr/lib/jvm/jdk1.8.0_231\" >> /opt/hadoop/etc/hadoop/hadoop-env.sh"

                sh "/opt/hadoop/sbin/start-dfs.sh"
            }
        }
    }
}
