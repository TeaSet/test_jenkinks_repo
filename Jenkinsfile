#!/usr/bin/env groovy

def buildUnix(label, stashName) {
	node(label) {
		checkout scm
		sh '''#!/bin/sh
			gcc hello.c -o hello_${label}.out
		'''
		stash name: stashName, includes: "*.out" 
		deleteDir()
	}
}

timestamps {
	stage('build') {
		parallel(
			"build_win": {
				node("windows") {
					checkout scm
					String vsvars_bat = 'Microsoft Visual Studio 12.0\\VC\\vcvarsall.bat' 
					bat """
						call "%ProgramFiles(X86)%\\${vsvars_bat}" x86
						cl.exe hello.c
					"""
					 stash name: "build_win", includes: "*.exe"
					 deleteDir()
				}
			},
			"build_mac": {
				buildUnix("mac", "build_mac")
			},
			"build_linux": {
				buildUnix("linux", "build_linux")
			}
		)
	}

	stage('store') {
		node("master") {
			unstash "build_mac"
			unstash "build_win"
			unstash "build_linux"
			archiveArtifacts artifacts: '*.*', onlyIfSuccessful: true
			deleteDir()
		}
	}
}