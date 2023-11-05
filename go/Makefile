isuports: test go.mod go.sum *.go cmd/isuports/*
	go build -o isuports ./cmd/isuports

test:
	go test -v ./...

SERVICE_NAME = isuports.service # systemctl status $(SERVICE_NAME)
BUILD_TARGET = isuports
ALP_REGEXP = '/api/organizer/player/[0-9a-z\-]+,/api/player/player/[0-9a-z\-]+'

restart: $(BUILD_TARGET) log.rotate restart-mysql restart-app restart-nginx # set build target on top
init: init.config init.asdf init.tools log.init
survey: survey.slowlog survey.nginx

restart-app:
	rm -f /tmp/webapp.sock
	sudo systemctl restart $(SERVICE_NAME)
	sudo systemctl status $(SERVICE_NAME) | tail -n 5

restart-nginx:
	sudo rsync -av ../conf/nginx/ /etc/nginx/
	sudo nginx -s reload
	sudo systemctl status nginx | tail -n 5

restart-mysql:
	sudo rsync -av ../conf/mysql/ /etc/mysql/
	sudo systemctl restart mysql.service
	sudo systemctl status mysql.service | tail -n 5

init.config:
	mkdir ../conf
	mkdir ../conf/nginx
	mkdir ../conf/mysql
	mkdir /home/isucon/webapp/conf/nginx/sites-enabled
	mkdir /home/isucon/webapp/conf/mysql/conf.d
	cp /etc/nginx/nginx.conf /home/isucon/webapp/conf/nginx/nginx.conf
	cp /etc/nginx/sites-enabled/* /home/isucon/webapp/conf/nginx/sites-enabled/
	cp /etc/mysql/conf.d/*.cnf /home/isucon/webapp/conf/mysql/conf.d/
	cp /etc/mysql/conf.d/*.cnf /home/isucon/webapp/conf/mysql/conf.d/

init.asdf:
	git clone https://github.com/asdf-vm/asdf.git ~/.asdf
	echo '. "/home/isucon/.asdf/asdf.sh"' > ~/.bashrc
	. ~/.bashrc

init.tools:
	sudo apt update
	sudo apt-get install -y percona-toolkit unzip
	asdf plugin add alp
	asdf install alp `asdf list-all alp | tail -1`
	asdf global alp `asdf list-all alp | tail -1`

survey.slowlog:
	sudo pt-query-digest /var/log/mysql/mysql-slow.log | \
	tee ../survey/slowlog/digest_`date +%Y%m%d%H%M%S`.txt

# need to customize regexp
survey.nginx:
	sudo cat /var/log/nginx/access.log | \
	alp json --sort=sum -r -m $(ALP_REGEXP)| \
	tee ../survey/nginx/alp_`date +%Y%m%d%H%M%S`.txt

log.init:
	sudo touch /var/log/mysql/mysql-slow.log
	sudo touch /var/log/nginx/access.log
	mkdir ../survey
	mkdir ../survey/slowlog
	mkdir ../survey/nginx


log.rotate:
	sudo mv /var/log/mysql/mysql-slow.log /var/log/mysql/mysql-slow.log.`date +%Y%m%d%H%M%S`
	sudo touch /var/log/mysql/mysql-slow.log
	sudo chmod 777 /var/log/mysql/mysql-slow.log
	sudo mv /var/log/nginx/access.log /var/log/nginx/access.log.`date +%Y%m%d%H%M%S`
	sudo touch /var/log/nginx/access.log

clean.all: clean.log
	rm -rf ../conf
	sudo rm -rf /home/isucon/.asdf

clean.log:
	rm -rf ../survey