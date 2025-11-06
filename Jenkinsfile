pipeline {

	agent { label 'built-in' }

	environment {

		CONTAINER_NAME = "ci-nginx-test-${BUILD_NUMBER}"
		HOST_PORT      = 9889
		LOCAL_FILE     = "index.html"
	}

	triggers {
		possSCM('H/2 * * * *')
	}

	stages {

		stage('Start Nginx Container') {
			steps {
				echo "Starting Nginx container ${CONTAINER_NAME}"
				echo "Mapping host port ${HOST_PORT} to container port 80."
				sh """
					docker run -d \
					-p ${HOST_PORT}:80 \
					-v ${WORKSPACE}/${LOCAL_FILE}:/usr/share/nginx/html/index.html:ro \
					nginx:latest
				"""
				echo "Waiting 5 seconds for Nginx to start..."
				sh "sleep 5"
			}
		}

		stage('Run CI Checks') {
			steps {
				script {
					echo "Checking HTTP status code..."
					def httpStatus = sh(
						script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${HOST_PORT}",
						returnStdout: true
					).trim()

					echo "HTTP Status Code: ${httpStatus}"
					if (httpStatus != "200") {
						error("Check Failed: Expected HTTP 200, but got ${httpStatus}")
					}
					echo "HTTP Status 200 OK."

					echo "Checking MD5 checksum..."

					def localMd5 = sh(
						script: "md5sum ${LOCAL_FILE} | awk '{print \$1}'",
					).trim()

					echo "Local MD5: ${localMd5}"
					echo "Remote MD5: ${remoteMd5}"

					if (localMd5 != remoteMd5) {
						error("Check Failed: MD5 sums do not match!")
					}
					echo "MD5 Sums Match."
				}
			}
		}
	}

	post {
		always {
			echo "Cleaning up container ${CONTAINER_NAME}..."
			sh "docker stop ${CONTAINDER_NAME} || true"
			sh "docker rm ${CONTAINER_NAME} || true"
		}

		failure {
			echo "Build failed.Sending notification..."
			emailext to: 'm.v.orekhov@gmail.com',
			         subject: "Build failded: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
			         body: """
					The pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER} didn't pass the checkout.
					Check the logs: ${env.BUILD_URL}
			         """
		}

		success {
			echo "Build successful. All checks passed."
		}
	}
}
