- server: [donweb](https://donweb.com) vps
- dns: [donweb](https://donweb.com)
- domain: [nic.ar](https://nic.ar/)
- os: [debian](https://www.debian.org/)
- service supervision: [daemontools](https://cr.yp.to/daemontools.html)
- http: [ucspi-tcp](https://cr.yp.to/ucspi-tcp.html) and [publicfile](https://cr.yp.to/publicfile.html)
- tls: [caddy](https://caddyserver.com/docs/) reverse proxy
- writing: [markdown](https://kristaps.bsd.lv/lowdown/) and html
- site generator: [ssg](https://romanzolotarev.com/ssg.html)
- deployment: [git](https://git-scm.com/)

## deployment

	local changes -> commit -> push to server -> post-receive hook -> deploy script -> ssg.sh

Post receive hook:

	#!/bin/sh
	
	echo $name (date) >> ~/commits
	$deployrepo/deploy

Deploy script:

	#!/bin/sh
	
	unset GIT_DIR
	
	wd=$deploydir
	repo=$wd/ncb.ar
	files=$repo/$filesdir
	name=ncb.ar
	url=ncb.ar
	out=/public/web/file/0
	
	echo $(date) >> $wd/last-commit.txt
	
	cd $repo
	git pull --verbose
	
	./ssg.sh $files $out $name $url

