颜色

```bash
_err(){
  echo -e "\033[1;31m[ERROR] $@\033[0m" >&2
}

_info(){
  echo -e "\033[1;32m[INFO] $@\033[0m" >&2
}
```

安装 `JeMalloc` 内存分配器

```bash
install_jemalloc(){
  cd /opt && wget $URL -O jemalloc.tar.bz2 && yum install -y bzip2
  tar jxvf jemalloc.tar.bz2 && cd jemalloc
  ./configura /opt/jemalloc
  make -j
  make install
  touch /etc/ld.so.conf.d/local.conf
  echo '/opt/jemalloc/lib' >> /etc/ld.so.conf.d/local.conf
  ldconfig -v|grep jemalloc
}
```

给用户添加 `sudo` 权限

```bash
add_user01_sudo(){
  echo "Defaults !requiretty
  user01 ALL=(root) NOPASSWD: /usr/bin/cat,/usr/bin/jstat" > /etc/sudoers.d/user01
}
```
