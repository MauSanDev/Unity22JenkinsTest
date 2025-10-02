pipeline {
    agent {
        label 'unity-2022.3.62f1-android-3'
    }

    parameters {
        choice(
            name: 'BUILD_TARGET',
            choices: ['Android', 'WebGL'],
            description: 'Platform to build for'
        )
        booleanParam(
            name: 'DEVELOPMENT_BUILD',
            defaultValue: false,
            description: 'Create development build'
        )
        booleanParam(
            name: 'GENERATE_ADDRESSABLES',
            defaultValue: true,
            description: 'Generate Addressable assets'
        )
        booleanParam(
            name: 'FORCE_CLEAN_BUILD',
            defaultValue: false,
            description: 'Force clean Library folder (troubleshooting only - slow!)'
        )
        string(
            name: 'BUILD_VERSION',
            defaultValue: '',
            description: 'Override build version (leave empty for auto: 1.0.BUILD_NUMBER)'
        )
        string(
            name: 'BUILD_SUFFIX',
            defaultValue: '',
            description: 'Build suffix identifier (leave empty for default: jenkins)'
        )
    }

    environment {
        UNITY_PROJECT_PATH = "${WORKSPACE}"
        UNITY_EMAIL = credentials('tga-unity-email')
        UNITY_PASSWORD = credentials('tga-unity-password')
        UNITY_SERIAL = credentials('tga-unity-serial')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out Unity 6 project..."
                checkout scm
            }
        }

        stage('Force Clean') {
            when {
                expression { params.FORCE_CLEAN_BUILD == true }
            }
            steps {
                echo "⚠️  FORCE CLEAN: This will significantly slow down the build!"
                sh 'rm -rf Library'
                echo "Library folder cleaned (expect slower build time)"
            }
        }


        stage('Build Android') {
            when {
                expression { params.BUILD_TARGET == 'Android' }
            }
            steps {
                script {
                    def buildVersion = params.BUILD_VERSION ?: "1.0.${BUILD_NUMBER}"
                    def buildSuffix = params.BUILD_SUFFIX ?: "jenkins"
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

                    sh """
                        set -x
                        echo "🚀 Starting Android build with JIT bypass..."
                        echo "Build Version: ${buildVersion}"
                        echo "Build Suffix: ${buildSuffix}"
                        echo "Commit Hash: ${commitHash}"
                        echo "Development Build: ${params.DEVELOPMENT_BUILD}"
                        echo "Generate Addressables: ${params.GENERATE_ADDRESSABLES}"
                        echo "⚠️  Using Mono interpreter mode to bypass JIT crashes"

                        export MONO_ENV_OPTIONS="--interpreter"

                        unity-editor \\
                            -batchmode \\
                            -quit \\
                            -nographics \\
                            -projectPath "${UNITY_PROJECT_PATH}" \\
                            -executeMethod BuildScript.BuildBatchMode \\
                            -logFile /dev/stdout \\
                            -buildTarget Android \\
                            -buildVersion "${buildVersion}" \\
                            -buildSuffix "${buildSuffix}" \\
                            -commitHash "${commitHash}" \\
                            -buildId "${BUILD_NUMBER}" \\
                            -buildOutputPath "Builds" \\
                            -generateAddressables ${params.GENERATE_ADDRESSABLES} \\
                            -developmentBuild ${params.DEVELOPMENT_BUILD} 2>&1 | tee android-build.log

                        UNITY_EXIT_CODE=\${PIPESTATUS[0]}
                        if [ \$UNITY_EXIT_CODE -ne 0 ]; then
                            echo "❌ Unity build failed with exit code \$UNITY_EXIT_CODE"
                            exit \$UNITY_EXIT_CODE
                        fi
                    """
                }
            }
        }

        stage('Build WebGL') {
            when {
                expression { params.BUILD_TARGET == 'WebGL' }
            }
            steps {
                script {
                    def buildVersion = params.BUILD_VERSION ?: "1.0.${BUILD_NUMBER}"
                    def buildSuffix = params.BUILD_SUFFIX ?: "jenkins"
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

                    sh """
                        set -x
                        echo "🌐 Starting WebGL build with JIT bypass..."
                        echo "Build Version: ${buildVersion}"
                        echo "Build Suffix: ${buildSuffix}"
                        echo "Commit Hash: ${commitHash}"
                        echo "Development Build: ${params.DEVELOPMENT_BUILD}"
                        echo "Generate Addressables: ${params.GENERATE_ADDRESSABLES}"
                        echo "⚠️  Using Mono interpreter mode to bypass JIT crashes"

                        export MONO_ENV_OPTIONS="--interpreter"

                        unity-editor \\
                            -batchmode \\
                            -quit \\
                            -nographics \\
                            -projectPath "${UNITY_PROJECT_PATH}" \\
                            -executeMethod BuildScript.BuildBatchMode \\
                            -logFile /dev/stdout \\
                            -buildTarget WebGL \\
                            -buildVersion "${buildVersion}" \\
                            -buildSuffix "${buildSuffix}" \\
                            -commitHash "${commitHash}" \\
                            -buildId "${BUILD_NUMBER}" \\
                            -buildOutputPath "Builds" \\
                            -generateAddressables ${params.GENERATE_ADDRESSABLES} \\
                            -developmentBuild ${params.DEVELOPMENT_BUILD} 2>&1 | tee webgl-build.log

                        UNITY_EXIT_CODE=\${PIPESTATUS[0]}
                        if [ \$UNITY_EXIT_CODE -ne 0 ]; then
                            echo "❌ Unity build failed with exit code \$UNITY_EXIT_CODE"
                            exit \$UNITY_EXIT_CODE
                        fi
                    """
                }
            }
        }
    }

    post {
        always {
            // Archive Unity build logs
            archiveArtifacts artifacts: '*.log', allowEmptyArchive: true, fingerprint: true

            // Archive build artifacts and reports
            archiveArtifacts artifacts: '**/Development/**/*', allowEmptyArchive: true
            archiveArtifacts artifacts: '**/QA/**/*', allowEmptyArchive: true
            archiveArtifacts artifacts: '**/Release/**/*', allowEmptyArchive: true
            archiveArtifacts artifacts: '**/BuildReport.json', allowEmptyArchive: true
            archiveArtifacts artifacts: '**/BuildParameters.json', allowEmptyArchive: true

            echo "Build completed on Unity 6 persistent agent with Builder system"
            echo "📥 Log files archived: android-build.log, webgl-build.log (if generated)"
        }
        success {
            echo "✅ Build successful! Check organized build folders and reports."
        }
        failure {
            echo "❌ Build failed - check logs above and BuildReport.json for details"
        }
    }
}