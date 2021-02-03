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
                    sh "pgrep -f redis"
                }
            }
        }
        stage('Publish') {
            steps {
                sh "cp /usr/bin/redis-server /srv/redis-server"
                // make a tarball for pick up
                sh "tar czvf /srv/redis_compile_build.tgz /srv/workspace/jpegleg-repo_redis_compile_main/ && touch /srv/redis_compile_pickup.lock"              
                sh "mkdir -p /srv/debbuild/redis_compile-1.0.0/DEBIAN/; mkdir -p /srv/debbuild/redis_compile-1.0.0/usr/bin/ >/dev/null"
                sh "cp /srv/workspace/jpegleg-repo_redis_compile_main/redis_compile.control /srv/debbuild/redis_compile-1.0.0/DEBIAN/control"
                sh "cp /srv/workspace/jpegleg-repo_redis_compile_main/postinst /srv/debbuild/redis_compile-1.0.0/DEBIAN/postinst"
                sh "chmod +x /srv/debbuild/redis_compile-1.0.0/DEBIAN/postinst"
             
                sh "chmod +x /srv/debbuild/redis_compile-1.0.0/usr/bin/redis-server"
                sh "cp  /usr/bin/redis-server /srv/debbuild/redis_compile-1.0.0/usr/bin/redis-server"
                sh "cd /srv/debbuild/ && tar czvf redis_compile-1.0.0.tar.gz redis_compile-1.0.0/ && dpkg -b ./redis_compile-1.0.0 ./redis_compile-1.0.0.deb"
            }
            post {
                success {
                    sh "ls /srv/redis_compile2_build.tgz || exit 1"
                    sh "cp /srv/debbuild/*.deb /srv/ && touch /srv/redis_compile_pickup.lock"
                }
            }
        }
        stage('deb Tests') {
            steps {
                // test the deb
                sh "bash /srv/debuild redis_compile-1.0.0.deb"
            }
            post {
                success {
                    sh "ls -larth /usr/bin/redis-server || exit 1"
                    sh "echo redis_compile > /srv/net-g.sums.txt"
                    sh "sha256sum /srv/redis_compile-1.0.0.deb >> /srv/redis_compile.sums.txt"
                    sh "sha1sum /srv/redis_compile-1.0.0.deb >> /srv/redis_compile.sums.txt"
                    sh "md5sum /srv/redis_compile-1.0.0.deb >> /srv/redis_compile.sums.txt"
                }
            }
        }
    }
}
