pipeline {
  agent any
  parameters {
    string(name: 'ApplicationScope', defaultValue: 'Admin Test APP', description: 'Name of the LifeTime application to deploy.')
    string(name: 'TriggeredBy', defaultValue: 'N/A', description: 'Name of LifeTime user that triggered the pipeline remotely.')
  }
  options { skipStagesAfterUnstable() }
  environment {
    ArtifactsFolder = "Artifacts"
    LifeTimeHostname = 'oslifetime.persistentproducts.com'
    LifeTimeAPIVersion = 2
    AuthorizationToken = credentials('LifeTimeServiceAccountToken')
    DevelopmentEnvironment = 'Development'
    RegressionEnvironment = 'UAT'
    AcceptanceEnvironment = 'UAT'
    PreProductionEnvironment = 'UAT'
    ProductionEnvironment = 'UAT'
    OSPackageVersion = '0.8.0'
  }

  
      stages {
              stage('Install Python Dependencies') {
      steps {
        
        // Only the virtual environment needs to be installed at the system level
        bat script: 'python -m pip install -q -I virtualenv --user', label: 'Install Python virtual environments'
        // Install the rest of the dependencies at the environment level and not the system level
        withPythonEnv('python') {
        bat script: "python -m pip install -U outsystems-pipeline==\"${env.OSPackageVersion}\"", label: 'Install required packages'
        }
      }
    }
    stage('Get and Deploy Latest Tags') {
      steps {
        script {
          bat script: 'python --version'
          bat script: 'C:/ProgramData/Jenkins/.jenkins/workspace/AdminTest-CDPipeline/.pyenv-python/Scripts/python -m pip install --upgrade setuptools', label: 'Upgrade setuptools'
          bat script: 'C:/ProgramData/Jenkins/.jenkins/workspace/AdminTest-CDPipeline/.pyenv-python/Scripts/python -m pip install --upgrade pip', label: 'Upgrade pip'
          bat script: 'C:/ProgramData/Jenkins/.jenkins/workspace/AdminTest-CDPipeline/.pyenv-python/Scripts/python -m pip install xunitparser==1.3.4', label: 'Install xunitparser'
          echo "Pipeline run triggered remotely by '${params.TriggeredBy}' for the following application: '${params.ApplicationScope}'"
          bat script: "mkdir ${env.ArtifactsFolder}", label: 'Create artifacts folder'
          bat script: 'python -m pip install -q -I virtualenv --user', label: 'Install Python virtual environments'
          withPythonEnv('python') {
            bat script: "python -m pip install -U outsystems-pipeline==\"${env.OSPackageVersion}\"", label: 'Install required packages'
            bat script: "python -m outsystems.pipeline.fetch_lifetime_data --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion}", label: 'Retrieve list of Environments and Applications'
            lock('deployment-plan-REG') {
              bat script: "python -m outsystems.pipeline.deploy_latest_tags_to_target_env --artifacts \"${env.ArtifactsFolder}\" --lt_url ${env.LifeTimeHostname} --lt_token ${env.AuthorizationToken} --lt_api_version ${env.LifeTimeAPIVersion} --source_env \"${env.DevelopmentEnvironment}\" --destination_env \"${env.RegressionEnvironment}\" --app_list \"${params.ApplicationScope}\"", label: "Deploy latest application tags to ${env.RegressionEnvironment}"
            }
          }
        }
      }
      post {
        always {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "*.cache", onlyIfSuccessful: true
            archiveArtifacts artifacts: "*_data/*.cache", onlyIfSuccessful: true
            stash name: "manifest",includes: "deployment_data/deployment_manifest.cache", allowEmpty: true
          }
        }
        failure {
          dir ("${env.ArtifactsFolder}") {
            archiveArtifacts artifacts: "DeploymentConflicts"
          }
        }
        cleanup {
          dir ("${env.ArtifactsFolder}") {
            deleteDir()
          }
        }
      }
    }
  }
  }
 
