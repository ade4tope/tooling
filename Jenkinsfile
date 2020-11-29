pipeline {
  agent any

      environment 
    {
        PROJECT     = 'zooto-tooling-prod'
        ECRURL      = '059636857273.dkr.ecr.eu-central-1.amazonaws.com'
        DEPLOY_TO = 'development'
    }

  stages {

    // stage("Initial cleanup") {
    //     steps {
    //     dir("${WORKSPACE}") {
    //         deleteDir()
    //     }
    //     }
    // }

    stage('Checkout Application Code')
    {
      steps {
            // echo 'Prepare Working directory'
            // script {
            //     sh("pwd && mkdir tmp") 
            // }
      checkout([
        $class: 'GitSCM', 
        doGenerateSubmoduleConfigurations: false, 
        extensions: [[$class: 'RelativeTargetDirectory',
             relativeTargetDir: 'tooling']],
        submoduleCfg: [], 
        // branches: [[name: '$branch']],
        userRemoteConfigs: [[url: "https://github.com/darey-devops/tooling.git ",credentialsId:'GITHUB_CREDENTIALS']] 	
        ])
        
      }
        }


// checkout([$class: 'GitSCM', branches: [[name: '*/' + env.BRANCH_NAME]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: false, recursiveSubmodules: true, reference: '', trackingSubmodules: false]], 
// submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/IshentRas/cookbook-openshift3']]])




        stage('Checkout Flux Deployment Repo For Release')
        {
        steps {
            //     echo 'Move out of working directory'
            // script {
            //     sh("pwd && mkdir helm") 
            // }
        checkout([
            $class: 'GitSCM', 
            doGenerateSubmoduleConfigurations: false, 
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'helm-tooling-frontend', 
                        $class: 'SubmoduleOption', disableSubmodules: false, parentCredentials: false, recursiveSubmodules: true, reference: '', 
                        trackingSubmodules: false]],
            // extensions: [[$class: 'RelativeTargetDirectory', 
            //     relativeTargetDir: 'helm']],
            submoduleCfg: [[url: "https://github.com/darey-devops/helm-tooling-frontend.git",credentialsId:'GITHUB_CREDENTIALS']], 
            branches: [[name: 'master']],
            userRemoteConfigs: [[url: "https://gitlab.com/zooto.io/fluxcd-deployments.git",credentialsId:'GIT_CREDENTIALS']]
            ])
            
        }
            }










          stage('Build preparations')
        {
            steps
            {
                script 
                {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                }
            }
        }

    stage('Build For Dev Environment') {
               when { branch pattern: "^feature.*|^bug.*|^dev", comparator: "REGEXP"}
            
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-central-1 | sed 's|https://||')") 
                // sh "docker build --network=host -t $IMAGE -f deploy/docker/Dockerfile ."
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("dev-$BUILD_NUMBER")
            }
            }
        }
      }

    stage('Build For Staging Environment') {
               when {
                expression { BRANCH_NAME ==~ /(staging|master|main)/ }
            }
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-central-1 | sed 's|https://||')") 
                // sh "docker build --network=host -t $IMAGE -f deploy/docker/Dockerfile ."
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("staging-$BUILD_NUMBER")
            }
            }
        }
    }


    stage('Build For Production Environment') {
        when { tag "release-*" }
        steps {
            echo 'Build Dockerfile....'
            script {
                sh("eval \$(aws ecr get-login --no-include-email --region eu-central-1 | sed 's|https://||')") 
                // sh "docker build --network=host -t $IMAGE -f deploy/docker/Dockerfile ."
                sh "docker build --network=host -t $IMAGE ."
                docker.withRegistry("https://$ECRURL"){
                docker.image("$IMAGE").push("prod-$BUILD_NUMBER")
            }
            }
        }
    }

//








      // stage('Update Helm appVersion') {
      //   steps {
      //       echo 'Update appVersion'
      //       sh '''
      //             cat helm/Chart.yaml.template | sed "s/appVersion: .*/appVersion: \"${BUILD_NUMBER}\"/g" > helm/Chart.yaml
      //         '''
      //   }
      // }
    }

        post
    {
        always
        {
            sh "docker rmi -f $IMAGE "
        }
    }
} 