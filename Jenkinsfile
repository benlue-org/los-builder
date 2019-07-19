node('swarm') {  
    stage('Preparation') {
        sh ''' set -x
            mkdir -p ~/bin                        
            curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
            chmod a+x ~/bin/repo
            PATH=~/bin:$PATH
            repo init -u https://github.com/LineageOS/android.git -b lineage-15.1
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
