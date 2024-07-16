@Library('my-shared-library') _

pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'HarisAmjad158')
    }

    stages {
        stage('Create Jenkins Node with WebSocket') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    def nodeConfig = """
                    import jenkins.model.*
                    import hudson.model.*
                    import hudson.slaves.*
                    import jenkins.slaves.*
                    import jenkins.slaves.RemotingWorkDirSettings

                    def instance = Jenkins.getInstance()

                    def nodeName = "new-node"
                    def nodeDescription = "This is a new node"
                    def remoteFS = "/home/jenkins"
                    def numExecutors = 2
                    def nodeMode = Node.Mode.NORMAL
                    def labelString = "agent"
                    def launcher = new JNLPLauncher(null, null)
                    launcher.setWebSocket(true)  // Enable WebSocket
                    def retentionStrategy = new RetentionStrategy.Always()

                    // WorkDirSettings to disable work directory
                    def workDirSettings = new RemotingWorkDirSettings.WorkDirSettings(null, true, null)

                    def node = new DumbSlave(
                        nodeName,
                        nodeDescription,
                        remoteFS,
                        numExecutors,
                        nodeMode,
                        labelString,
                        launcher,
                        retentionStrategy,
                        new LinkedList()
                    )

                    node.setWorkDirSettings(workDirSettings)

                    instance.addNode(node)
                    """

                    // Execute the node configuration
                    evaluate(nodeConfig)
                }
            }
        }
        stage('Git Checkout') {
            when { expression { params.action == 'create' } }
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/harisamjad0158/Java_app_3.0.git"
                )
            }
        }
        stage('Unit Test maven') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    mvnTest()
                }
            }
        }
        stage('Integration Test maven') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    mvnIntegrationTest()
                }
            }
        }
        stage('Static code analysis: Sonarqube') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    statiCodeAnalysis(SonarQubecredentialsId)
                }
            }
        }
        stage('Quality Gate Status Check: Sonarqube') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    QualityGateStatus(SonarQubecredentialsId)
                }
            }
        }
        stage('Maven Build: maven') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    mvnBuild()
                }
            }
        }
        stage('Docker Image Build') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerBuild("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }
        stage('Docker Image Scan: trivy') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerImageScan("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }
        stage('Docker Image Push: DockerHub') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }
        stage('Docker Image Cleanup: DockerHub') {
            when { expression { params.action == 'create' } }
            steps {
                script {
                    dockerImageCleanup("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }
    }
}
