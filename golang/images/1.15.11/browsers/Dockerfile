###
### DO NOT MODIFY THIS FILE.  THIS FILE HAS BEEN AUTOGENERATED
###

FROM circleci/golang:1.15.11

#
# Install Java 11 LTS / OpenJDK 11
#
RUN if grep -q Debian /etc/os-release && grep -q stretch /etc/os-release; then \
		echo 'deb http://deb.debian.org/debian stretch-backports main' | sudo tee -a /etc/apt/sources.list.d/stretch-backports.list; \
	elif grep -q Ubuntu /etc/os-release && grep -q xenial /etc/os-release; then \
		sudo apt-get update && sudo apt-get install -y software-properties-common && \
		sudo add-apt-repository -y ppa:openjdk-r/ppa; \
	fi && \
	sudo apt-get update && sudo apt-get install -y openjdk-11-jre openjdk-11-jre-headless openjdk-11-jdk openjdk-11-jdk-headless && \
	sudo apt-get install -y bzip2 libgconf-2-4 # for extracting firefox and running chrome, respectively

# install firefox
#
RUN FIREFOX_URL="https://download.mozilla.org/?product=firefox-latest-ssl&os=linux64&lang=en-US" \
  && ACTUAL_URL=$(curl -Ls -o /dev/null -w %{url_effective} $FIREFOX_URL) \
  && curl --silent --show-error --location --fail --retry 3 --output /tmp/firefox.tar.bz2 $ACTUAL_URL \
  && sudo tar -xvjf /tmp/firefox.tar.bz2 -C /opt \
  && sudo ln -s /opt/firefox/firefox /usr/local/bin/firefox \
  && sudo apt-get install -y libgtk3.0-cil-dev libasound2 libasound2 libdbus-glib-1-2 libdbus-1-3 \
  && rm -rf /tmp/firefox.* \
  && firefox --version

# install geckodriver

RUN export GECKODRIVER_LATEST_RELEASE_URL=$(curl https://api.github.com/repos/mozilla/geckodriver/releases/latest | jq -r ".assets[] | select(.name | test(\"linux64.tar.gz$\")) | .browser_download_url") \
     && curl --silent --show-error --location --fail --retry 3 --output /tmp/geckodriver_linux64.tar.gz "$GECKODRIVER_LATEST_RELEASE_URL" \
     && cd /tmp \
     && tar xf geckodriver_linux64.tar.gz \
     && rm -rf geckodriver_linux64.tar.gz \
     && sudo mv geckodriver /usr/local/bin/geckodriver \
     && sudo chmod +x /usr/local/bin/geckodriver \
     && geckodriver --version

# install chrome

RUN cd /home/circleci && wget -O google-chrome-stable_current_amd64.deb -t 5 "https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb" \
    && (sudo dpkg -i google-chrome-stable_current_amd64.deb || sudo apt-get -fy install)  \
    && rm -rf google-chrome-stable_current_amd64.deb \
    && sudo sed -i 's|HERE/chrome"|HERE/chrome" --disable-setuid-sandbox --no-sandbox|g' \
        "/opt/google/chrome/google-chrome" \
    && google-chrome --version

RUN CHROME_VERSION="$(google-chrome --version)" \
    && export CHROMEDRIVER_RELEASE="$(echo $CHROME_VERSION | sed 's/^Google Chrome //')" && export CHROMEDRIVER_RELEASE=${CHROMEDRIVER_RELEASE%%.*} \
    && CHROMEDRIVER_VERSION=$(curl --silent --show-error --location --fail --retry 4 --retry-delay 5 http://chromedriver.storage.googleapis.com/LATEST_RELEASE_${CHROMEDRIVER_RELEASE}) \
    && curl --silent --show-error --location --fail --retry 4 --retry-delay 5 --output /tmp/chromedriver_linux64.zip "http://chromedriver.storage.googleapis.com/$CHROMEDRIVER_VERSION/chromedriver_linux64.zip" \
    && cd /tmp \
    && unzip chromedriver_linux64.zip \
    && rm -rf chromedriver_linux64.zip \
    && sudo mv chromedriver /usr/local/bin/chromedriver \
    && sudo chmod +x /usr/local/bin/chromedriver \
    && chromedriver --version

# Google Chrome < v85 had libxss1 as a dependency. It no longer does but people
# have been depending on it so installing manually.
RUN sudo apt-get install -y libxss1

# start xvfb automatically to avoid needing to express in circle.yml
ENV DISPLAY :99
RUN printf '#!/bin/sh\nXvfb :99 -screen 0 1280x1024x24 &\nexec "$@"\n' > /tmp/entrypoint \
  && chmod +x /tmp/entrypoint \
        && sudo mv /tmp/entrypoint /docker-entrypoint.sh

# ensure that the build agent doesn't override the entrypoint
LABEL com.circleci.preserve-entrypoint=true

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["/bin/sh"]
