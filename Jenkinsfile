void resetSourceTree() {
  echo 'Reseting source tree...'
  dir(env.SOURCE_DIR) {
    sh '''#!/usr/bin/env bash
    repo forall -c "git reset --hard"'''
  }
}

int cleanUp() {
  echo 'Cleaning up environment...'
  dir(env.SOURCE_DIR) {
    ansiColor('xterm') {
      return sh (returnStatus: true, script: '''#!/usr/bin/env bash
      # Load build environment
      . build/envsetup.sh
      lunch $LUNCH
      # Clean up PRODUCT directory keeping the common stuff
      # mkdir -p $OUT
      # rm -rf $(cd $OUT/../;pwd)
      # make clean
      # Clean up temporary incoming directory
      rm -rf $INCOMING_TMP_DIR
      ''')
    }
  }
}

void repoPickGerritChanges() {
  echo 'Applying gerrit changes...'
  dir(env.SOURCE_DIR) {
    return sh (returnStatus: true, script: '''#!/usr/bin/env bash
      function check_gerrit_picks_result {
        if [ "0" -ne "$?" ]
        then
          echo "Gerrit picks failed."
          exit 1
        fi
      }
      if [ ! -z "$GERRIT_CHANGES" ]
      then
        IS_HTTP=$(echo $GERRIT_CHANGES | grep http)
        if [ -z "$IS_HTTP" ]
        then
          python ./vendor/unlegacy/build/tools/repopick.py $GERRIT_CHANGES
          check_gerrit_picks_result
        else
          python ./vendor/unlegacy/build/tools/repopick.py $(curl $GERRIT_CHANGES)
          check_gerrit_picks_result
        fi
      fi''')
  }
}

int build(String buildTargets) {
  echo 'Building android...'
  env.BUILD_TARGETS = buildTargets
  dir(env.SOURCE_DIR) {
    ansiColor('xterm') {
      return sh (returnStatus: true, script: '''#!/usr/bin/env bash
      # Make sure ccache is in PATH
      export PATH="$PATH:/opt/local/bin/:$PWD/prebuilts/misc/$(uname|awk '{print tolower($0)}')-x86/ccache"
      # Load build environment
      . build/envsetup.sh
      # Try to lunch
      lunch $LUNCH
      if [ "0" -ne "$?" ]
      then
        echo "Lunch failed."
        exit 1
      fi
      # Setup ccache size
      #if [ ! "$(ccache -s|grep -E 'max cache size'|awk '{print $4}')" = "100.0" ]
      #then
      #  ccache -M 100G
      #fi
      # Check if we need to cleanup the OUT directory
      LAST_CLEAN=0
      if [ -f .clean ]
      then
        LAST_CLEAN=$(date -r .clean +%s)
      fi
      TIME_SINCE_LAST_CLEAN=$(expr $(date +%s) - $LAST_CLEAN)
      # Convert this to hours
      TIME_SINCE_LAST_CLEAN=$(expr $TIME_SINCE_LAST_CLEAN / 60 / 60)
      if [ $TIME_SINCE_LAST_CLEAN -gt "24" -o $CLEAN = "true" ]
      then
        echo "Cleaning!"
        touch .clean
        make clean
      else
        echo -e "Skipping full clean: $TIME_SINCE_LAST_CLEAN hours since last clean.\nCleaning PRODUCT directory..."
        mkdir -p $OUT
        rm -rf $(cd $OUT/../;pwd)
      fi
      OTATOOLS_TARGET=otatools
      time make -j$JOBS $BUILD_TARGETS $OTATOOLS_TARGET
      if [ "0" -ne "$?" ]
      then
        echo "Build failed."
        exit 2
      fi
      ''')
    }
  }
}

int createOtaPackage(String otaType) {
  echo 'Creating OTA Package...'
  env.OTA_TYPE = otaType
  dir(env.SOURCE_DIR) {
    ansiColor('xterm') {
      return sh (returnStatus: true, script: '''#!/usr/bin/env bash
      export OTA_EXIT_CODE=0
      # Load build environment
      . build/envsetup.sh
      lunch $LUNCH
      cp -f $OUT/boot.img ${ARCHIVE_DIR}/. || export OTA_EXIT_CODE=3
      cp -f $OUT/recovery.img ${ARCHIVE_DIR}/.  || export OTA_EXIT_CODE=4
      if [ "${BOOT_AND_RECOVERY_IMAGES_ONLY}" == "true" ]
      then
        # Clean up PRODUCT directory keeping the common stuff
        mkdir -p $OUT
        rm -rf $(cd $OUT/../;pwd)
        exit $OTA_EXIT_CODE
      fi
      export DEVICE_TARGET_FILES_PATH=$DEVICE_TARGET_FILES_DIR/$(date -u +%Y%m%d-%H%M).zip
      mkdir -p $DEVICE_TARGET_FILES_DIR
      cp ${OUT}/obj/PACKAGING/target_files_intermediates/*target_files*.zip $DEVICE_TARGET_FILES_PATH
      rm -f $(readlink ${DEVICE_TARGET_FILES_DIR}/last.zip) 2>/dev/null
      rm -f ${DEVICE_TARGET_FILES_DIR}/last.zip 2>/dev/null
      rm -f ${DEVICE_TARGET_FILES_DIR}/last.prop 2>/dev/null
      mv ${DEVICE_TARGET_FILES_DIR}/latest.zip ${DEVICE_TARGET_FILES_DIR}/last.zip 2>/dev/null
      mv ${DEVICE_TARGET_FILES_DIR}/latest.prop ${DEVICE_TARGET_FILES_DIR}/last.prop 2>/dev/null
      ln -sf $DEVICE_TARGET_FILES_PATH ${DEVICE_TARGET_FILES_DIR}/latest.zip
      cp -f $OUT/system/build.prop ${DEVICE_TARGET_FILES_DIR}/latest.prop
      cp -f $OUT/system/build.prop ${ARCHIVE_DIR}/build.prop
      export PLATFORM_VERSION=`grep ro.build.version.release $DEVICE_TARGET_FILES_DIR/latest.prop | cut -d '=' -f2`
      export OUTPUT_FILE_NAME=${BUILD_PRODUCT}_${DEVICE}-${PLATFORM_VERSION}
      export LATEST_DATE=$(date -r $DEVICE_TARGET_FILES_DIR/latest.prop +%Y%m%d-%H%M)
      export OTA_OPTIONS="-v -p $ANDROID_HOST_OUT $OTA_COMMON_OPTIONS"
      export OTA_INC_OPTIONS="$OTA_OPTIONS $OTA_INC_EXTRA_OPTIONS"
      export OTA_FULL_OPTIONS="$OTA_OPTIONS $OTA_FULL_EXTRA_OPTIONS"
      export OTA_INC_FAILED="false"
      if [ "${MARK_AS_EXPERIMENTAL}" == "true" ] || [ ! -z "$GERRIT_CHANGES" ]
      then
        if [ ! -z "$EXPERIMENTAL_TAG" ]
        then
          export OUTPUT_FILE_NAME=${OUTPUT_FILE_NAME}-${EXPERIMENTAL_TAG}
        else
          export OUTPUT_FILE_NAME=${OUTPUT_FILE_NAME}-EXPERIMENTAL
        fi
      fi
      if [ -f ${DEVICE_TARGET_FILES_DIR}/last.zip ] && [ "${OTA_TYPE}" == "incremental" ]
      then
        export LAST_DATE=$(date -r $DEVICE_TARGET_FILES_DIR/last.prop +%Y%m%d-%H%M)
        export FILE_NAME=${OUTPUT_FILE_NAME}-${LAST_DATE}-TO-${LATEST_DATE}
        ./build/tools/releasetools/ota_from_target_files \
          $OTA_INC_OPTIONS \
          --incremental_from $DEVICE_TARGET_FILES_DIR/last.zip \
          $DEVICE_TARGET_FILES_DIR/latest.zip $ARCHIVE_DIR/$FILE_NAME.zip || export OTA_INC_FAILED="true" && export OTA_EXIT_CODE=2
        if [ -s $ARCHIVE_DIR/$FILE_NAME.zip ] || [ "${OTA_INC_FAILED}" == "true" ]
        then
          export FILE_NAME=${OUTPUT_FILE_NAME}-${LATEST_DATE}
          ./build/tools/releasetools/ota_from_target_files \
            $OTA_FULL_OPTIONS \
            $DEVICE_TARGET_FILES_DIR/latest.zip $ARCHIVE_DIR/$FILE_NAME.zip || export OTA_EXIT_CODE=1
        else
          rm -f $ARCHIVE_DIR/$FILE_NAME.*
        fi
      else
        export FILE_NAME=${OUTPUT_FILE_NAME}-${LATEST_DATE}
        ./build/tools/releasetools/ota_from_target_files \
          $OTA_FULL_OPTIONS \
          $DEVICE_TARGET_FILES_DIR/latest.zip $ARCHIVE_DIR/$FILE_NAME.zip || export OTA_EXIT_CODE=1
      fi
      for f in $(ls $ARCHIVE_DIR/*.zip*)
      do
        md5sum $f | cut -d ' ' -f1 > $ARCHIVE_DIR/$(basename $f).md5sum
      done
      # Clean up PRODUCT directory keeping the common stuff
      mkdir -p $OUT
      rm -rf $(cd $OUT/../;pwd)
      exit $OTA_EXIT_CODE
      ''')
    }
  }
}

int publishToPortal(String path) {
  echo 'Publishing OTA Package to builds portal...'
  env.OTA_URL_PATH = path
  dir(env.SOURCE_DIR) {
    ansiColor('xterm') {
      return sh (returnStatus: true, script: '''#!/usr/bin/env bash
      export PLATFORM_VERSION=`grep ro.build.version.release ${ARCHIVE_DIR}/build.prop | cut -d '=' -f2`
      export FILENAME=$(basename $(ls ${ARCHIVE_DIR}/${BUILD_PRODUCT}_${DEVICE}-${PLATFORM_VERSION}*.zip))
      export MD5SUM=$(cat ${ARCHIVE_DIR}/${BUILD_PRODUCT}_${DEVICE}-${PLATFORM_VERSION}*.zip.md5sum)
      export OTA_URL="${OTA_URL_PATH}/${FILENAME}"
      export TYPE=release
      if [ "${MARK_AS_EXPERIMENTAL}" == "true" ] || [ ! -z "$GERRIT_CHANGES" ]
      then
        export TYPE=test
      fi
      export POST_DATA="{ \\"device\\": \\"${DEVICE}\\", \\"filename\\": \\"${FILENAME}\\", \\"md5sum\\": \\"${MD5SUM}\\", \\"romtype\\": \\"${TYPE}\\", \\"url\\": \\"${OTA_URL}\\", \\"version\\": \\"${PLATFORM_VERSION}\\" }"
      echo "${POST_DATA}" > /tmp/postdata
      curl -X POST ${BUILDS_PORTAL_URL}/api/v1/add_build -H "apiKey:$API_KEY" -H "Content-Type:application/json" --data-binary "@/tmp/postdata" 2>/dev/null
      curl -X POST ${BUILDS_PORTAL_URL}/api/v1/purgecache -H "apiKey:$API_KEY" 2>/dev/null
      rm -f /tmp/postdata
      ''')
    }
  }
}

properties([parameters([choice(choices: ['jfltexx', 'jfvelte', 'cheeseburger', 'bacon', 'j5xnlte', 'hero2lte', 'herolte', ''], description: 'Select your device', name: 'device'),
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
			    sh 'mkdir -p ~/bin'
                	    sh 'curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo'
                            sh 'chmod a+x ~/bin/repo'
			}	
		}
		stage('Repo Sync'){
			dir("${WORKSPACE}/los-build/${params.branch}") {
			    sh '''#!/bin/bash
			       set -x
			       source ~/.profile
			       repo init -u "${MIRROR_PATH}" -b $branch
			       repo sync -f --force-sync --force-broken --no-clone-bundle --no-tags -j$(nproc --all)
			    '''		
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
