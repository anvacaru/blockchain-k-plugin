pipeline {
  agent {
    dockerfile {
      reuseNode true
    }
  }
  options {
    ansiColor('xterm')
  }
  stages {
    stage("Init title") {
      when { changeRequest() }
      steps {
        script {
          currentBuild.displayName = "PR ${env.CHANGE_ID}: ${env.CHANGE_TITLE}"
        }
      }
    }
    stage("Test compilation") {
      when { changeRequest() }
      steps {
        dir ('llvm-backend') {
          checkout([$class: 'GitSCM',
          branches: [[name: '*/master']],
          extensions: [[$class: 'SubmoduleOption',
                        disableSubmodules: false,
                        parentCredentials: false,
                        recursiveSubmodules: true,
                        reference: '',
                        trackingSubmodules: false]], 
          userRemoteConfigs: [[url: 'git@github.com:kframework/llvm-backend.git']]])
          sh '''
            mkdir build
            cd build
            cmake .. -DCMAKE_BUILD_TYPE=Release
            make include
          '''
        }
        dir ('libff') {
          checkout([$class: 'GitSCM',
          branches: [[name: '*/master']],
          extensions: [[$class: 'SubmoduleOption',
                        disableSubmodules: false,
                        parentCredentials: false,
                        recursiveSubmodules: true,
                        reference: '',
                        trackingSubmodules: false]], 
          userRemoteConfigs: [[url: 'git@github.com:scipr-lab/libff.git']]])
          sh '''
            mkdir build
            cd build
            cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=install
            make -j16
            make install
          '''
        }
        dir ('proxygen') {
          checkout([$class: 'GitSCM',
          branches: [[name: '*/master']],
          extensions: [[$class: 'SubmoduleOption',
                        disableSubmodules: false,
                        parentCredentials: false,
                        recursiveSubmodules: true,
                        reference: '',
                        trackingSubmodules: false]], 
          userRemoteConfigs: [[url: 'git@github.com:facebook/proxygen.git']]])
          sh '''
            cd proxygen
            ./build.sh -m
            ./install.sh
          '''
        }
        sh 'make -j16'
      }
    }
    stage('Deploy') {
      when { branch 'master' }
      steps {
        build job: 'rv-devops/master', propagate: false, wait: false                                  \
            , parameters: [ booleanParam(name: 'UPDATE_DEPS_SUBMODULE', value: true)                  \
                          , string(name: 'PR_REVIEWER', value: 'ehildenb')                            \
                          , string(name: 'UPDATE_DEPS_REPOSITORY', value: 'kframework/evm-semantics') \
                          , string(name: 'UPDATE_DEPS_SUBMODULE_DIR', value: 'deps/plugin')           \
                          ]
      }
    }
  }
}
