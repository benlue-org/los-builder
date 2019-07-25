def _pipelineNotify(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus =  buildStatus ?: 'SUCCESSFUL'

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"

    // Override default values based on build status
    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESSFUL') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }
}

properties([parameters([choice(choices: ['jfltexx', 'jfvelte', 'TL-WR1043NDv1'], description: 'Select your device to build', name: 'device'), 
			choice(choices: ['lineage-16.0', 'lineage-15.1', 'lineage-15.0', 'develop'], description: 'Select your Branch to buils', name: 'branch'), 
			choice(choices: ['test1', 'test2', 'test3', 'test4'], description: 'Treffe deine Auswahl für Build 3', name: 'test'),
			booleanParam(defaultValue: true, description: 'Willst du ein clean build?', name: 'make clean')]), [$class: 'JiraProjectProperty'], pipelineTriggers([[$class: 'PeriodicFolderTrigger', interval: '1d']
			])
])

node {
  try {
      _pipelineNotify()

	stage('Preperation') {
	sh 'mkdir -p ~/bin'
        sh 'curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo'
        sh 'chmod a+x ~/bin/repo'
	sh 'ln -sf /mnt/los-build ./los-build'
//        echo "Downloading ${params.device}.xml ..."      
  }
      stage('Repo Sync') {
	      withEnv(['MIRROR_PATH=/mnt/los-mirror/LineageOS/android.git',
       	 	       "BUILD_PATH=${env.WORKSPACE}/los-build",
	 	       'LOCAL_MANIFESTS=/home/lineageos/android/lineage/.repo/local_manifests']) {
	dir("${env.BUILD_PATH}"/${params.branch}) {
	  echo "${params.device}"
          echo "${params.branch}"
	  echo "${BUILD_PATH}"
	  sh 'ls -lah "${WORKSPACE}/los-build"'
	  //sh 'printenv'
	  sh 'ls -lah'
	  
	}
      }
 
//    stage('Update Feeds') {
        //sh "mv feeds.conf feeds.conf.old"
//      sh "wget https://raw.githubusercontent.com/benlue-org/openwrt-builder/master/feeds/feeds.conf"
//      sh label: 'Feeds Update', returnStdout: true, script: './scripts/feeds update -a'
//    }
     
//    stage ('Install Feeds') {
//	sh label: 'Feeds Install', returnStdout: true, script: './scripts/feeds install -a'
//        sh "rm -f .config"
//        sh "rm -f diffconfig"
//        sh "wget https://raw.githubusercontent.com/benlue-org/openwrt-builder/master/profiles/ar71xx/tlwdr4300v1/diffconfig"
//        sh "mv diffconfig .config"
//        sh "echo CONFIG_TARGET_ar71xx_generic_DEVICE_tl-wdr4300-v1=y"
//        sh "make defconfig"	  
//      }
      
//      stage('Build') {
        //sh label: 'Make Clean', returnStdout: true, script: 'make clean'
//        sh label: 'Build Process', returnStdout: true, script: 'make -j1 V=sc'
//      }
      
//      stage('Archive') {
//        archiveArtifacts 'bin/targets/**/**/*.bin'
//      }
  }
  }
  catch (e) {
      currentBuild.result = "FAILED"
      throw e
  }
  finally {
      _pipelineNotify(currentBuild.result)
  }
}
