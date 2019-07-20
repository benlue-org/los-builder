node('master') {  
    stage('Version Check') {
        sh ''' set -x
            cat /proc/version
        '''
    }
    stage('Apt Update') {
        sh ''' set -x
            apt update 2>/dev/null 
        '''
    }
    stage('Apt Upgrade') {
        sh ''' set -x
            apt upgrade -y 2>/dev/null
        '''
    }
    stage('Install Dep.') {
        sh ''' set -x
            apt-get install -y --no-install-recommends \
                  bc \
                  ssh \
                  bison \
                  build-essential \
                  ccache \
                  curl \
                  flex \
                  g++-multilib \
                  gcc-multilib \
                  git \
                  gnupg \
                  gperf \
                  imagemagick \
                  lib32ncurses5-dev \
                  lib32readline-dev \
                  lib32z1-dev \
                  liblz4-tool \
                  libncurses5-dev \
                  libsdl1.2-dev \
                  libssl-dev \
                  libwxgtk3.0-dev \
                  libxml2 \
                  libxml2-utils \
                  lzop \
                  pngcrush \
                  rsync \
                  schedtool \
                  squashfs-tools \
                  xsltproc \
                  zip \
		  gcc \
		  binutils \
		  bzip2 \
		  flex \
		  python \
		  perl \
		  make \
		  grep \
		  diffutils \
		  unzip \
		  gawk \
		  subversion \
                  zlib1g-dev 2>/dev/null
        '''
    }    
}
