#��������� ��������� ������

           yum install -y \
           redhat-lsb-core \
           wget \
           rpmdevtools \
           rpm-build \
           createrepo \
           yum-utils

           yum install -y pcre-devel gcc


#������ ����� NGINX � ������� ��� � ���������� openssl

           wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm


#��� ��������� ������ ������ � �������� ���������� ��������� ����� ��������� ��� ������

           rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm


#����� ������� � ��������������� ��������� ��� openssl - ��
#����������� ��� ������

           wget https://www.openssl.org/source/latest.tar.gz

           tar -xvf latest.tar.gz


#�������� ��� ����������� ���� � �������� ������ �� ���� ������

           yum-builddep /root/rpmbuild/SPECS/nginx.spec


#������ ��� spec ���� ���� NGINX ��������� � ������������ ���
�������

           nano /root/rpmbuild/SPECS/nginx.spec

 
#��������� 

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



#������ ����� ���������� � ������ RPM ������
  
           rpmbuild -bb rpmbuild/SPECS/nginx.spec    


#�������� ��� ������ ���������

           ll rpmbuild/RPMS/x86_64/

#�����

           -rw-r--r--. 1 root root 2003528 ��� 30 05:05 nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
           -rw-r--r--. 1 root root 2489280 ��� 30 05:05 nginx-debuginfo-1.14.1-1.el7_4.ngx.x86_64.rpm



#������ ����� ���������� ��� ����� � ��������� ��� nginx ��������

           yum localinstall /root/rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm

           systemctl start nginx
 
           systemctl status nginx


#������ ��������� � �������� ������ �����������.���������� ��� ������� � NGINX ��
#��������� /usr/share/nginx/html, �������� ��� ������� repo

           mkdir /usr/share/nginx/html/repo


#�������� ���� ��� ��������� RPM

           cp rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/


#RPM ��� ��������� ����������� Percona-Server

           wget https://downloads.percona.com/downloads/percona-release/percona-release-1.0-9/redhat/percona-release-1.0-9.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm


#�������������� ����������� ��������

           createrepo /usr/share/nginx/html/repo/


#��� ������������ �������� � NGINX ������ � �������� ��������.
#� location / � ����� /etc/nginx/conf.d/default.conf ������� ��������� autoindex on.

           nano /etc/nginx/conf.d/default.conf

#��������� ��������� � ������������� NGINX

           nginx -t

           nginx -s reload

#������ ����� ����������

          curl -a http://localhost/repo/


#��� ������ ��� ����, ���� �������������� �����������.
#������� ��� � /etc/yum.repos.d


           cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF

     

#�������� ��� ����������� ����������� � ��������� ��� � ��� ����

           yum repolist enabled | grep otus

           yum list | grep otus


#��� ��� NGINX � ��� ��� ����� ��������� ����������� percona-release

           yum install percona-release -y


#��� ������ �������. � ������ ���� ��� ����������� �������� ����������� (� ���
#�������� ��� ������ ���������� ������), ����� �� ��������� �������

           createrepo /usr/share/nginx/html/repo/
