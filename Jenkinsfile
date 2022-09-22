pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "Login_Cloud_Server", , usernameVariable: 'USERNAME')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    key: "-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAhwxBBdUTqPmpWxoh0Xe4WdLfRYM4iJo5pryT/M1yiPjuum7A
bt4vqrVlCmqR51tI41IvkgA6w3NS6go8MnFMdYUMnbOj7lltnhjOgdSUV8Qve7o5
1vsHZ1lZY1jdn0YxzkjJwTPXiqlgtpwP9Q21bidewTrpG5Jgi5mKT0P+IWn2wcxG
dWz9WszdEt7s6my7U1Splb/bBjxygoQ4xP6bafQDsB5xQ/6A6WmHRyMCYa2kXoHr
j8ShrZJvKQsn0+aZ/V+/cxiMIvVxft4ZOLo5s/ECbw6eixGyky07tASBFRchu7DJ
fa38ZSSgVeKjwfro9ycdfsAWTWZtxTw89eNU9QIDAQABAoIBAG9CiDtu3aCj94Pn
1p7FPGs8UNfrEONx9DdLO7zV4hu4wq1z2zQ79xd0FIdtX0E1MqqdpjVp3P/zfgb1
YbzJHQ3hDJDMVC1lHetXUqMh6QkZx2ju9wIHlITv1rYDm6rB4PyreRSkSlqhyt0H
XBovLh9PbkBR8YTWppW8bHd7c95AP9+PGYB6oimozAnYRwNocNzmW5K8PrJjdqfs
Hs+epHW8H/H+vpFbhmt94glxNl2FNUiS+nidiJSpV5J5lHWenGeEUikp1DDEvJCg
BxTerUKkBWjz3Bm3F1tnBQkoxrwlbmJvbfbF5ptkZ1l5TAnHMMwJ6NnXXIJWnMJj
fzs3beECgYEA4CLBsF9XbRYBrsFBW0oPwh0kvOX87pdSpd3OMzT2WYqKzCrFFemh
D+H4WTGi8th33dyetEAAbdIO4m5oVhSsnkf+ir+hYh46e+UAC0ytI7sS24+hq6bB
BBhmWx9qQBK/Uvhc8gLQYPC4BtpzelbyF9+mwmXsQgUVLW6T8Dw4CG0CgYEAmj82
xUsPCohc6G2dz7Ok5NFGoPAwFO0ZVM3G5vfHyyDfQ5A48m+QK5snIbswwff6yM7W
qSYSJ324gnIneSiWrABAcwm/rAh4uc5cYmsSu284XFJ6YEWuxzVKXzdr7Tmbgchx
vlmMfYalK3zwXH3bRfhJI3fEcR5TznN+Pa6ruakCgYEAsd+jav8e+LlgOHmyDmqm
Oty6DRdQNWDt/CgcvlKntsPWBtVid1NjuKESYGad9K+J4Q52/IFWVdFAcr5AGyBp
JWvpO998icuHik9gS5dcSGDsREamfPznbQKYKHSz84ltQMFNsdo92NDwmq++uTZL
Bls9kkUky/gQqG97BEomBbkCgYBXknwuFyc6+6CD9XgbbAq6Pnay+KrTtqkjFJFQ
oGy2TrtzSHaMbfqUR0o4RGayOXAQgh4teofkE+SlatouV3TzwlDU/zvrGAQyuY6J
8fB4qfR9tfX0optQTlkjJfwIeyRm0r6BK6YvvjoYLp7oZCwR1ZzwnhbRgj5if/+0
VW75wQKBgEPvqjhxvNoBcwgD3IMuJJPXRdKgqkw4IRK4LB4sH4EkuaY0mO/gbiut
yXJQuEesjoLerXVsAddvAh6RVlDGA991JmtvYYWvbxAm0PsTgX/L0cCIlA6MtgLk
4J6cj4fUQyRyGuR9/lAPMIyaAkk6kKk0qRkwIP+NOVfYRp3onRUH
-----END RSA PRIVATE KEY-----"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
