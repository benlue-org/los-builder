node('master') {  
    stage('Preparation') {
        sh ''' set -x
            cat /proc/version
            #apt update 2>/dev/null 
            #apt upgrade 2>/dev/null
            #apt-get install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev -y 2>/dev/null
            mkdir -p ~/bin                        
            curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
            chmod a+x ~/bin/repo
            PATH=~/bin:$PATH
            repo init -u https://github.com/LineageOS/android.git -b lineage-16.0
            '''
    }
    stage('Repo Sync') {
        sh '''#!/bin/bash
                       set -x
                       PATH=~/bin:$PATH
                       repo sync -f --force-sync --force-broken --no-clone-bundle --no-tags -j$(nproc --all)

                    '''

    }
}
