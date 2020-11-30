#скачиваем следующие пакеты

           yum install -y \
           redhat-lsb-core \
           wget \
           rpmdevtools \
           rpm-build \
           createrepo \
           yum-utils

           yum install -y pcre-devel gcc


#возмем пакет NGINX и соберем его с поддержкой openssl

           wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm


#при установке такого пакета в домашней директории создаетс€ древо каталогов дл€ сборки

           rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm


#нужно скачать и разархивировать исходники дл€ openssl - он
#потребуетс€ при сборке

           wget https://www.openssl.org/source/latest.tar.gz

           tar -xvf latest.tar.gz


#поставим все зависимости чтоб в процессе сборки не было ошибок

           yum-builddep /root/rpmbuild/SPECS/nginx.spec


#правим сам spec файл чтоб NGINX собиралс€ с необходимыми нам
опци€ми

           nano /root/rpmbuild/SPECS/nginx.spec

 
#результат 

           %build
           ./configure %{BASE_CONFIGURE_ARGS} \
              --with-cc-opt="%{WITH_CC_OPT}" \
              --with-ld-opt="%{WITH_LD_OPT}" \
              --with-openssl=/root/openssl-1.1.1h
           make %{?_smp_mflags}
           %{__mv} %{bdir}/objs/nginx \
               %{bdir}/objs/nginx-debug
           ./configure %{BASE_CONFIGURE_ARGS} \
               --with-cc-opt="%{WITH_CC_OPT}" \
               --with-ld-opt="%{WITH_LD_OPT}"



#“еперь можно приступить к сборке RPM пакета
  
           rpmbuild -bb rpmbuild/SPECS/nginx.spec    


#”бедимс€ что пакеты создались

           ll rpmbuild/RPMS/x86_64/

#вывод

           -rw-r--r--. 1 root root 2003528 но€ 30 05:05 nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
           -rw-r--r--. 1 root root 2489280 но€ 30 05:05 nginx-debuginfo-1.14.1-1.el7_4.ngx.x86_64.rpm



#“еперь можно установить наш пакет и убедитьс€ что nginx работает

           yum localinstall /root/rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm

           systemctl start nginx
 
           systemctl status nginx


#“еперь приступим к созданию своего репозитори€.ƒиректори€ дл€ статики у NGINX по
#умолчанию /usr/share/nginx/html, создадим там каталог repo

           mkdir /usr/share/nginx/html/repo


# опируем туда наш собранный RPM

           cp rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/


#RPM дл€ установки репозитори€ Percona-Server

           wget https://downloads.percona.com/downloads/percona-release/percona-release-1.0-9/redhat/percona-release-1.0-9.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm


#»нициализируем репозиторий командой

           createrepo /usr/share/nginx/html/repo/


#ƒл€ прозрачности настроим в NGINX доступ к листингу каталога.
#¬ location / в файле /etc/nginx/conf.d/default.conf добавим директиву autoindex on.

           nano /etc/nginx/conf.d/default.conf

#ѕровер€ем синтаксис и перезапускаем NGINX

           nginx -t

           nginx -s reload

#“еперь можно посмотреть

          curl -a http://localhost/repo/


#¬се готово дл€ того, чтоб протестировать репозиторий.
#ƒобавим его в /etc/yum.repos.d


           cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF

     

#”бедимс€ что репозиторий подключилс€ и посмотрим что в нем есть

           yum repolist enabled | grep otus

           yum list | grep otus


#“ак как NGINX у нас уже стоит установим репозиторий percona-release

           yum install percona-release -y


#¬се прошло успешно. ¬ случае если вам потребуетс€ обновить репозиторий (а это
#делаетс€ при каждом добавлении файлов), снова то выполните команду

           createrepo /usr/share/nginx/html/repo/
