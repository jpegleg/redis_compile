# redis_compile
A small utility for compiling redis from source with libc and tls and running the full tests.

Made for debian-based systems, such as a debian build/compile host or dev machine. Jenkinsfile example pipeline included included.

Example to download and compile in pwd:

chmod +x redis_compile redis-6.0.10.tar.gz | tee redis_compile_log.txt
