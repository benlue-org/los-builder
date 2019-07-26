properties([parameters([choice(choices: ['jfltexx', 'jfvelte', 'sunny'], description: 'Select your device', name: 'device'),
                        choice(choices: ['lineage-16.0', 'lineage-15.1', 'lineage-15.0', 'develop'], description: 'Select your branch', name: 'branch'),
                        choice(choices: ['test 1', 'test 2', 'test 3'], description: 'Select your test', name: 'test'),
                        booleanParam(defaultValue: true, description: 'Do you build clean?', name: 'make clean')]),
[$class: 'JiraProjectProperty']])

node('swarm') {
    withEnv(['MIRROR_PATH=/mnt/los-mirror/LineageOS/android.git',
	     'BUILD_PATH=${env.WORKSPACE}/los-build',
     	     'LOCAL_MANIFESTS=/home/lineageos/android/lineage/.repo/local_manifests']) {
			     
		stage('Preparation'){
			dir("${WORKSPACE}/los-build/${params.branch}") {
			    sh 'ln -sf /mnt/los-build ./los-build'
			}	
		}
		stage('Repo Sync'){
			dir("${WORKSPACE}/los-build/${params.branch}") {
				sh 'repo init -u ${MIRROR_PATH} -b ${params.branch}'
				sh 'ls -lah'
			}
		}
		stage('Patching Process'){
			dir("${WORKSPACE}/los-build/${params.branch}") {
				echo "Patching Process"
			}
		}
		stage('Build Process'){
			dir("${WORKSPACE}/los-build/${params.branch}") {
				echo "Build Process"
			}
		}
		stage('Revert Patch'){
			dir("${WORKSPACE}/los-build/${params.branch}") {
				echo "Revert Patch"
			}
		}
		stage('OTA Package'){
			dir("${WORKSPACE}/los-build/${params.branch}") {
				echo "OTA Package"
			}
		}
        	stage('Archiving') {
			dir("${WORKSPACE}/los-build/${params.branch}") {
				echo "Something"
			}
		}
        	stage('Publishing') {
			dir("${WORKSPACE}/los-build/${params.branch}") {
				echo "Something"
			}
		}
	}
}
