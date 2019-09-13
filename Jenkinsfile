import jenkins.model.Jenkins;
import java.text.SimpleDateFormat;
import java.nio.file.Files;
import java.nio.file.StandardOpenOption;

pipeline {
    agent any
	stages {
		stage('run speed test') {
			steps {
				sh 'docker run kklipsch/run-speedtest-cli | grep -E "Download:|Upload:"'
			}
		}

		stage('write results') {
			steps {
				script {
					def build = Jenkins.getInstance().getItemByFullName(env.JOB_NAME).getBuildByNumber(Integer.parseInt(env.BUILD_NUMBER));
					def logContent = build.logFile.text
						
					SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd  HH:mm:ss");
					String now = simpleDateFormat.format(new Date());
	                currentBuild.description = now;
	                
					String[] lines = logContent.split(System.getProperty("line.separator"));
					String graphData = "date,download,upload\n";
					graphData += now + ",";
					int i = 0;
					for (String line : lines) {
						if (line.startsWith("Download:") && i != 2) {
							graphData += line.split(":")[1] + ",";
							i++;
						} else if (line.startsWith("Upload:") && i != 2) {
							graphData += line.split(":")[1] + ",";
							i++;
						}
					}
					graphData += "\n";

					File output = new File(env.WORKSPACE, "data.csv");
					Files.write(output.toPath(), graphData.replaceAll("Mbit/s", "").replaceAll("Mbit/s", "").getBytes(), StandardOpenOption.CREATE);
				}
			}
		}
		
		stage('archive') {
			steps {
				archiveArtifacts 'data.csv'
			}
		}
	}
	
	post {
		always {
			plot csvFileName: 'plot-856fbd0f-69e7-44df-9867-8d8f598dabeb.csv', csvSeries: [[displayTableFlag: true, exclusionValues: 'downloads,uploads', file: 'data.csv', inclusionFlag: 'OFF', url: '']], group: 'speedtest', style: 'line', title: 'speedtest', yaxis: 'mbps', useDescr: true
			deleteDir()
		}
	}
}