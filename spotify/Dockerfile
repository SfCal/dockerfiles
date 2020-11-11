# Run spotify in a container
#
#docker run -d \
#	-v /etc/localtime:/etc/localtime:ro \
#	-v /tmp/.X11-unix:/tmp/.X11-unix \
#	-e "DISPLAY=unix${DISPLAY}" \
#	-e QT_DEVICE_PIXEL_RATIO \
#	--security-opt seccomp:unconfined \
#	--device /dev/snd \
#	--device /dev/dri \
#	--group-add audio \
#	--group-add video \
#	-v "$HOME/.spotify/cache:/home/spotify/.cache/spotify" \
#	-v "$HOME/.spotify/config:/home/spotify/.config/spotify" \
#	--name spotify \
#	${DOCKER_REPO_PREFIX}/spotify
#
FROM debian:sid-slim
LABEL maintainer "Jessie Frazelle <jess@linux.com>"

RUN	apt-get update && apt-get install -y \
	ca-certificates \
	curl \
	dirmngr \
	git \
	gnupg \
	--no-install-recommends \
 	&& curl -sS https://download.spotify.com/debian/pubkey_0D811D58.gpg | apt-key add -  \
	&& echo "deb http://repository.spotify.com stable non-free" >> /etc/apt/sources.list.d/spotify.list \
	&& apt-get update && apt-get install -y \
	alsa-utils \
	libgl1-mesa-dri \
	libgl1-mesa-glx \
	libpulse0 \
	libsm6 \
	spotify-client \
	xdg-utils \
	--no-install-recommends \
	&& rm -rf /var/lib/apt/lists/*


ENV HOME /home/spotify
RUN useradd --create-home --home-dir $HOME spotify \
	&& mkdir -p /home/spotify/.config/spotify \
	&& touch /home/spotify/.config/spotify/prefs \
	&& mkdir -p /home/spotify/.config/spicetify/Backup \
# above dirs needed for spicetify to create configs
	&& gpasswd -a spotify audio \
	&& chown -R spotify:spotify $HOME  \
# spicetify shouldn't be run as root and it needs access to spotify
	&& chmod 777 /usr/share/spotify \
	&& chmod 777 /usr/share/spotify/Apps -R

WORKDIR $HOME
USER spotify

# spicetify specific setup
RUN  curl -fsSL https://raw.githubusercontent.com/khanhas/spicetify-cli/master/install.sh | sh \
#	&& /home/spotify/spicetify-cli/spicetify config prefs_path "/home/spotify/.config/spotify/prefs" \
	&& /home/spotify/spicetify-cli/spicetify \
	&& /home/spotify/spicetify-cli/spicetify backup apply enable-devtool \
	&& git clone https://github.com/morpheusthewhite/spicetify-themes \
	&& cd spicetify-themes \
	&& cp -r * ~/.config/spicetify/Themes \
	&& cd ~/.config/spicetify/Themes/Dribbblish \
	&& cp dribbblish.js ../../Extensions \
	&& /home/spotify/spicetify-cli/spicetify config extensions dribbblish.js \
	&& /home/spotify/spicetify-cli/spicetify config current_theme Dribbblish color_scheme base \
	&& /home/spotify/spicetify-cli/spicetify config inject_css 1 replace_colors 1 overwrite_assets 1 \
	&& /home/spotify/spicetify-cli/spicetify apply

ENTRYPOINT	[ "spotify", "--force-device-scale-factor=2" ]
