## docker-rclone-mount

![docker-rclone](https://pbs.twimg.com/media/EPvel23XUAEtz7a?format=png&name=small "docker-rclone")


using an existing rclone.conf or setting up a new remote, you can run this container to mount your remote filesystem and share it with the host and other containers

* [rclone](https://rclone.org)
* [encfs](https://github.com/vgough/encfs)

shamelessly coppied from [tynor88](https://github.com/tynor88/docker-rclone-mount)

## Configuring rclone.conf

### Copying existing conf
	mkdir p $(pwd)/config/rclone-mount
	cp ~/.config/rclone/rclone.conf $(pwd)/config/rclone-mount/.rclone.conf

### Creating a new conf
	docker run -it --rm \
		-v $(pwd)/config/rclone-mount:/config \
		pknw1/rclone-mount rclone --config="/config/.rclone.conf" config

## Running a configured instance
### docker-cli

	REMOTE=yourremotename
	docker run -d \
		--name=rclone-$REMOTE \
		-v $(pwd)/config/rclone-$REMOTE:/config \
		--cap-add SYS_ADMIN \
		--device /dev/fuse \
		--security-opt apparmor:unconfined \
		-e RCLONE_REMOTE_MOUNT=$REMOTE:/ \
		-e PUID=666 \
		-e PGID=666 \
		-e TZ=Europe/London \
		-v $(pwd)/config/rclone-$REMOTE:/config \
		-v /mnt/rclone-$REMOTE:/data:shared \
		-v /etc/localtime:/etc/localtime \
		-v /tmp:/tmp \
		--restart=unless-stopped \
		pknw1/rclone-mount

### docker-compose

	version: '3'
	services:
	  rclone-ovh:
	    image: pknw1/rclone-mount
	    container_name: rclone-mount
	    security_opt:
	      - apparmor:unconfined
	    cap_add:
	      - SYS_ADMIN
	    devices:
	      - "/dev/fuse:/dev/fuse"
	    environment:
	      - PUID=666
	      - PGID=666
	      - PUMASK=0000
	      - TZ=Europe/London
	      - RCLONE_REMOTE_MOUNT=remote:/
	    volumes:
	      - ./config/rclone-mount:/config
	      - /mnt/rclone-remote:/data:shared
	      - /tmp:/tmp
	      - /etc/localtime:/etc/localtime
	    restart: unless-stopped

## Further development

* encfs mounting
* automatically recognise encrypted volumes and mount depending on ENCFS_PASS environment variable
