pipeline {
    agent {
        label 'deb'
    }


    stages {
        stage('Fetch') {
            steps {
                // Get some code from a GitHub repository
                sh "cd /srv; rm -rf redis_compile"
                sh "rm -rf redis_compile; git clone https://github.com/jpegleg/redis_compile"
            }
            post {
                success {
                    sh "ls redis_compile"
                }
            }
        }
        stage('Execute Build') {
            steps {
                sh "cd redis_compile && chmod +x redis_compile && ./redis_compile redis-6.0.10.tar.gz"
            }
            post {
                success {
                    sh "ls /usr/bin/redisrolling"
                }
            }
        }
        stage('Execute systemd setup') {
            steps {
                sh "cp /srv/workspace/jpegleg-repo_redis_compile_main/redisrolling.service /etc/systemd/system/multi-user.target.wants/redisrolling.service"
                sh "systemctl disable redis 2>/dev/null"
                sh "systemctl enable redisrolling"
                sh "systemctl restart redisrolling 2>/dev/null"
            }
            post {
                success {
                    sh "pgrep -f redisrolling"
                }
            }
        }
        stage('Publish') {
            steps {
                sh "cp /usr/bin/redis-server /srv/redis-server"
                // make a tarball for pick up
                sh "tar czvf /srv/redis_compile_build.tgz /srv/workspace/jpegleg-repo_redis_compile_main/ && touch /srv/redis_compile_pickup.lock"
                sh "mkdir -p /srv/debbuild/redisrolling-1.0.0/DEBIAN/; mkdir -p /srv/debbuild/redisrolling-1.0.0/etc/systemd/system/multi-user.target.wants/ 2>/dev/null; mkdir -p /srv/debbuild/redisrolling-1.0.0/usr/bin/ >/dev/null; mkdir /srv/debbuild/redisrolling-1.0.9/etc >/dev/null"
                sh "cp /srv/workspace/jpegleg-repo_redis_compile_main/redis_compile.control /srv/debbuild/redisrolling-1.0.0/DEBIAN/control"
                sh "cp /srv/workspace/jpegleg-repo_redis_compile_main/postinst /srv/debbuild/redisrolling-1.0.0/DEBIAN/postinst"
                sh "cp /srv/workspace/jpegleg-repo_redis_compile_main/redisrolling.conf /srv/debbuild/redisrolling-1.0.0/etc/redisrolling.conf"
                sh "cp /srv/workspace/jpegleg-repo_redis_compile_main/redisrolling.service /etc/systemd/system/multi-user.target.wants/redisrolling.service"
                sh "chmod +x /srv/debbuild/redisrolling-1.0.0/DEBIAN/postinst"
                sh "cp  /usr/bin/redisrolling /srv/debbuild/redisrolling-1.0.0/usr/bin/redisrolling"
                sh "chmod +x /srv/debbuild/redisrolling-1.0.0/usr/bin/redisrolling"
                sh "cd /srv/debbuild/ && tar czvf redisrolling-1.0.0.tar.gz redisrolling-1.0.0/ && dpkg -b ./redisrolling-1.0.0 ./redisrolling-1.0.0.deb"
            }
            post {
                success {
                    sh "ls /srv/redis_compile_build.tgz || exit 1"
                    sh "cp /srv/debbuild/*.deb /srv/ && touch /srv/redis_compile_pickup.lock"
                }
            }
        }
        stage('deb Tests') {
            steps {
                // test the deb
                sh "bash /srv/debuild redisrolling-1.0.0.deb"
            }
            post {
                success {
                    sh "ls -larth /usr/bin/redis-server || exit 1"
                    sh "echo redis_compile > /srv/redis_compile.sums.txt"
                    sh "sha256sum /srv/redisrolling-1.0.0.deb >> /srv/redis_compile.sums.txt"
                    sh "sha1sum /srv/redisrolling-1.0.0.deb >> /srv/redis_compile.sums.txt"
                    sh "md5sum /srv/redisrolling-1.0.0.deb >> /srv/redis_compile.sums.txt"
                }
            }
        }
    }
}
