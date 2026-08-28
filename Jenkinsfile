pipeline {

    agent any

    options {
        retry(conditions: [nonresumable()], count: 2)
        durabilityHint('PERFORMANCE_OPTIMIZED')
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {

        // ============================================================
        // JAVA / MAVEN
        // ============================================================

        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-17.0.2'
        MAVEN_HOME = 'D:\\Softwarepath\\apache-maven-3.8.5'

        // ============================================================
        // PROJECT
        // ============================================================

        PROJECT_DIR = '.'

        // Actual artifact from pom.xml
        APP_JAR = 'target\\maker-checker-banking-0.0.1-SNAPSHOT.jar'

        // ============================================================
        // BACKEND
        // ============================================================

        BACKEND_PORT = '8000'

        // Spring Boot application runs on port 8000
        BACKEND_URL = 'http://localhost:8000'

        // ============================================================
        // FRONTEND WAR
        //
        // IMPORTANT:
        // This WAR is NOT taken from Git.
        // Jenkins will use the WAR already present
        // on the local Windows machine.
        // ============================================================

        APPZILLON_SERVER_WAR = 'C:\\deploy\\maker-checker-frontend\\AppzillonServer.war'
        CHECKER_MAKER_WAR = 'C:\\deploy\\maker-checker-frontend\\Checker_Maker.war'

        // Context names used when copying WARs into Tomcat webapps
        APPZILLON_CONTEXT = 'AppzillonServer'
        CHECKER_MAKER_CONTEXT = 'Checker_Maker'

        // ============================================================
        // TOMCAT
        // ============================================================

        TOMCAT_HOME = 'D:\\Softwarepath\\apache-tomcat-9.0.53'
        TOMCAT_PORT = '8080'

        // ============================================================
        // FRONTEND
        // ============================================================

        FRONTEND_URL = 'http://localhost:8080/Checker_Maker/'

        // ============================================================
        // PLAYWRIGHT
        //
        // Change this if your Playwright project is elsewhere.
        // ============================================================

       // PLAYWRIGHT_DIR = 'D:\\playwright-maker-checker'
    }


    stages {


        // ============================================================
        // 1. CHECKOUT
        // ============================================================

        stage('Checkout') {

            steps {

                echo '=========================================='
                echo 'CHECKOUT MAKER-CHECKER PROJECT'
                echo '=========================================='

                checkout scm

                bat '''
                    @echo off

                    echo Current workspace:
                    cd

                    echo.
                    echo Workspace contents:
                    dir

                    echo.
                    echo Git status:
                    git status
                '''
            }
        }


        // ============================================================
        // 2. VERIFY ENVIRONMENT
        // ============================================================

        stage('Verify Environment') {

            steps {

                echo '=========================================='
                echo 'VERIFYING ENVIRONMENT'
                echo '=========================================='

                bat '''
                    @echo off

                    set "JAVA_HOME=%JAVA_HOME%"
                    set "PATH=%JAVA_HOME%\\bin;%MAVEN_HOME%\\bin;%PATH%"

                    echo.
                    echo JAVA_HOME:
                    echo %JAVA_HOME%

                    echo.
                    echo MAVEN_HOME:
                    echo %MAVEN_HOME%

                    echo.
                    echo JAVA VERSION:
                    java -version

                    echo.
                    echo MAVEN VERSION:
                    mvn -version

                    echo.
                    echo Checking project...

                    if not exist "%WORKSPACE%\\%PROJECT_DIR%\\pom.xml" (

                        echo ERROR:
                        echo pom.xml not found.

                        echo.
                        echo Expected:
                        echo %WORKSPACE%\\%PROJECT_DIR%\\pom.xml

                        exit /b 1
                    )

                    echo.
                    echo pom.xml found successfully.
                '''
            }
        }


        // ============================================================
        // 3. BUILD BACKEND
        // ============================================================

        stage('Build Backend') {

            steps {

                echo '=========================================='
                echo 'BUILDING MAKER-CHECKER BACKEND'
                echo '=========================================='

                bat '''
                    @echo off

                    set "JAVA_HOME=%JAVA_HOME%"
                    set "PATH=%JAVA_HOME%\\bin;%MAVEN_HOME%\\bin;%PATH%"

                    cd /d "%WORKSPACE%\\%PROJECT_DIR%"

                    echo.
                    echo Current directory:
                    cd

                    echo.
                    echo Starting Maven build...

                    mvn clean package -DskipTests

                    if errorlevel 1 (

                        echo.
                        echo ==========================================
                        echo MAVEN BUILD FAILED
                        echo ==========================================

                        exit /b 1
                    )

                    echo.
                    echo ==========================================
                    echo MAVEN BUILD SUCCESSFUL
                    echo ==========================================

                    echo.
                    echo Target directory:

                    dir target
                '''
            }
        }


        // ============================================================
        // 4. VERIFY JAR
        // ============================================================

        stage('Verify Backend JAR') {

            steps {

                echo '=========================================='
                echo 'VERIFYING BACKEND JAR'
                echo '=========================================='

                bat '''
                    @echo off

                    cd /d "%WORKSPACE%\\%PROJECT_DIR%"

                    echo Expected JAR:
                    echo %APP_JAR%

                    if not exist "%APP_JAR%" (

                        echo.
                        echo ==========================================
                        echo ERROR: JAR NOT FOUND
                        echo ==========================================

                        echo.
                        echo Target directory contents:

                        dir target

                        exit /b 1
                    )

                    echo.
                    echo ==========================================
                    echo BACKEND JAR FOUND
                    echo ==========================================

                    dir "%APP_JAR%"
                '''
            }
        }


        // ============================================================
        // 5. VERIFY FRONTEND WAR
        // ============================================================

        stage('Verify Appzillon WAR Files') {

            steps {

                echo '=========================================='
                echo 'VERIFYING LOCAL APPZILLON WAR FILES'
                echo '=========================================='

                bat '''
                    @echo off

                    echo AppzillonServer.war:
                    echo %APPZILLON_SERVER_WAR%

                    if not exist "%APPZILLON_SERVER_WAR%" (

                        echo.
                        echo ==========================================
                        echo ERROR: APPZILLON SERVER WAR NOT FOUND
                        echo ==========================================

                        echo.
                        echo Jenkins is expecting the WAR at:

                        echo %APPZILLON_SERVER_WAR%

                        echo.
                        echo Please make sure the WAR exists
                        echo on the Jenkins machine.

                        exit /b 1
                    )

                    echo.
                    echo ==========================================
                    echo FRONTEND WAR FOUND
                    echo ==========================================

                    dir "%APPZILLON_SERVER_WAR%"

                    echo.
                    echo Checker_Maker.war:
                    echo %CHECKER_MAKER_WAR%

                    if not exist "%CHECKER_MAKER_WAR%" (

                        echo ERROR: CHECKER MAKER WAR NOT FOUND

                        echo Jenkins is expecting the WAR at:
                        echo %CHECKER_MAKER_WAR%

                        exit /b 1
                    )

                    dir "%CHECKER_MAKER_WAR%"
                '''
            }
        }


        // ============================================================
        // 6. STOP EXISTING BACKEND
        // ============================================================

        stage('Stop Backend') {

            steps {

                echo '=========================================='
                echo 'STOPPING EXISTING BACKEND'
                echo '=========================================='

                bat '''
                    @echo off

                    echo Checking port %BACKEND_PORT%...

                    for /f "tokens=5" %%a in ('netstat -ano ^| findstr :%BACKEND_PORT% ^| findstr LISTENING') do (

                        echo Stopping process PID %%a

                        taskkill /F /PID %%a >nul 2>&1
                    )

                    echo.
                    echo Backend stop completed.

                    ping 127.0.0.1 -n 3 >nul
                '''
            }
        }


        // ============================================================
        // 7. START BACKEND
        // ============================================================

        stage('Start Backend') {

            steps {

                echo '=========================================='
                echo 'STARTING MAKER-CHECKER BACKEND'
                echo '=========================================='

                bat '''
                    @echo off

                    set "JAVA_HOME=%JAVA_HOME%"
                    set "PATH=%JAVA_HOME%\\bin;%PATH%"

                    cd /d "%WORKSPACE%\\%PROJECT_DIR%"

                    echo.
                    echo Starting:

                    echo java -jar %APP_JAR%

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                        "$env:JENKINS_NODE_COOKIE = 'dontKillMe'; ^
                         $env:JENKINS_SERVER_COOKIE = 'dontKillMe'; ^
                         $java = Join-Path $env:JAVA_HOME 'bin\\java.exe'; ^
                         $process = Start-Process -FilePath $java -ArgumentList @('-jar','%APP_JAR%') -WorkingDirectory '%WORKSPACE%\\%PROJECT_DIR%' -RedirectStandardOutput '%WORKSPACE%\\%PROJECT_DIR%\\backend.log' -RedirectStandardError '%WORKSPACE%\\%PROJECT_DIR%\\backend-error.log' -WindowStyle Hidden -PassThru; ^
                         if ($process.HasExited) { exit $process.ExitCode }"

                    echo.
                    if errorlevel 1 (
                        echo Backend process could not be started.
                        exit /b 1
                    )

                    echo Backend startup command executed.

                    echo.
                    echo Waiting for backend to initialize...

                    ping 127.0.0.1 -n 8 >nul

                    echo.
                    echo ==========================================
                    echo BACKEND LOG
                    echo ==========================================

                    if exist backend.log (

                        powershell -Command "Get-Content backend.log -Tail 50"

                    ) else (

                        echo backend.log not found
                    )

                    if exist backend-error.log (
                        echo.
                        echo Backend error log:
                        type backend-error.log
                    )
                '''
            }
        }


        // ============================================================
        // 8. BACKEND HEALTH CHECK
        // ============================================================

        stage('Backend Health Check') {

            steps {

                echo '=========================================='
                echo 'BACKEND HEALTH CHECK'
                echo '=========================================='

                bat '''
                    @echo off

                    set RETRIES=20

                    :CHECK_BACKEND

                    echo.
                    echo Checking backend port %BACKEND_PORT%...
                    echo Attempts remaining: %RETRIES%

                    netstat -ano | findstr :%BACKEND_PORT% | findstr LISTENING >nul

                    if not errorlevel 1 (

                        echo.
                        echo ==========================================
                        echo BACKEND IS RUNNING
                        echo ==========================================

                        echo.
                        echo Backend:
                        echo %BACKEND_URL%

                        exit /b 0
                    )

                    set /a RETRIES-=1

                    if %RETRIES% LEQ 0 (

                        echo.
                        echo ==========================================
                        echo BACKEND FAILED TO START
                        echo ==========================================

                        echo.
                        echo Port status:

                        netstat -ano | findstr :%BACKEND_PORT%

                        echo.
                        echo Backend log:

                        if exist "%WORKSPACE%\\%PROJECT_DIR%\\backend.log" (

                            type "%WORKSPACE%\\%PROJECT_DIR%\\backend.log"

                        ) else (

                            echo backend.log not found
                        )

                        exit /b 1
                    )

                    echo Backend not ready.

                    ping 127.0.0.1 -n 4 >nul

                    goto CHECK_BACKEND
                '''
            }
        }


        // ============================================================
        // 9. STOP TOMCAT
        // ============================================================

        stage('Stop Tomcat') {

            steps {

                echo '=========================================='
                echo 'STOPPING TOMCAT'
                echo '=========================================='

                bat '''
                    @echo off

                    if not exist "%TOMCAT_HOME%\\bin\\shutdown.bat" (

                        echo ERROR:
                        echo Tomcat shutdown.bat not found.

                        echo %TOMCAT_HOME%

                        exit /b 1
                    )

                    echo Calling Tomcat shutdown...

                    call "%TOMCAT_HOME%\\bin\\shutdown.bat"

                    echo.
                    echo Waiting for Tomcat to stop...

                    ping 127.0.0.1 -n 6 >nul

                    echo.
                    echo Checking remaining Tomcat process...

                    for /f "tokens=5" %%a in ('netstat -ano ^| findstr :%TOMCAT_PORT% ^| findstr LISTENING') do (

                        echo Killing remaining Tomcat PID %%a

                        taskkill /F /PID %%a >nul 2>&1
                    )

                    ping 127.0.0.1 -n 3 >nul

                    echo.
                    echo Tomcat stopped.
                '''
            }
        }


        // ============================================================
        // 10. DEPLOY FRONTEND WAR
        // ============================================================

        stage('Deploy Appzillon WAR Files') {

            steps {

                echo '=========================================='
                echo 'DEPLOYING APPZILLON WAR FILES TO TOMCAT'
                echo '=========================================='

                bat '''
                    @echo off

                    echo.
                    echo Tomcat:
                    echo %TOMCAT_HOME%

                    echo.
                    echo AppzillonServer.war:
                    echo %APPZILLON_SERVER_WAR%

                    echo Checker_Maker.war:
                    echo %CHECKER_MAKER_WAR%


                    if not exist "%TOMCAT_HOME%\\webapps" (

                        echo ERROR:
                        echo Tomcat webapps directory not found.

                        exit /b 1
                    )


                    echo.
                    echo ==========================================
                    echo REMOVING OLD FRONTEND
                    echo ==========================================

                    for %%C in ("%APPZILLON_CONTEXT%" "%CHECKER_MAKER_CONTEXT%") do (
                        rmdir /S /Q "%TOMCAT_HOME%\\webapps\\%%~C" >nul 2>&1
                        del /F /Q "%TOMCAT_HOME%\\webapps\\%%~C.war" >nul 2>&1
                    )


                    echo.
                    echo ==========================================
                    echo COPYING FRONTEND WAR
                    echo ==========================================

                    copy /Y "%APPZILLON_SERVER_WAR%" "%TOMCAT_HOME%\\webapps\\%APPZILLON_CONTEXT%.war"

                    if errorlevel 1 (
                        echo APPZILLON SERVER WAR COPY FAILED
                        exit /b 1
                    )

                    copy /Y "%CHECKER_MAKER_WAR%" "%TOMCAT_HOME%\\webapps\\%CHECKER_MAKER_CONTEXT%.war"


                    if errorlevel 1 (

                        echo.
                        echo ==========================================
                        echo FRONTEND WAR COPY FAILED
                        echo ==========================================

                        exit /b 1
                    )


                    echo.
                    echo ==========================================
                    echo FRONTEND WAR DEPLOYED
                    echo ==========================================

                    dir "%TOMCAT_HOME%\\webapps\\%APPZILLON_CONTEXT%.war"
                    dir "%TOMCAT_HOME%\\webapps\\%CHECKER_MAKER_CONTEXT%.war"
                '''
            }
        }


        // ============================================================
        // 11. START TOMCAT
        // ============================================================

        stage('Start Tomcat') {

            steps {

                echo '=========================================='
                echo 'STARTING TOMCAT'
                echo '=========================================='

                bat '''
                    @echo off

                    set "JAVA_HOME=%JAVA_HOME%"
                    set "PATH=%JAVA_HOME%\\bin;%PATH%"

                    set "CATALINA_HOME=%TOMCAT_HOME%"
                    set "JENKINS_NODE_COOKIE=dontKillMe"

                    echo JAVA_HOME:
                    echo %JAVA_HOME%

                    echo.
                    echo CATALINA_HOME:
                    echo %CATALINA_HOME%

                    echo.
                    echo Starting Tomcat...

                    call "%TOMCAT_HOME%\\bin\\catalina.bat" start

                    echo.
                    echo Tomcat start command executed.

                    echo.
                    echo Waiting for Tomcat...

                    ping 127.0.0.1 -n 15 >nul


                    echo.
                    echo ==========================================
                    echo TOMCAT PORT CHECK
                    echo ==========================================

                    netstat -ano | findstr :%TOMCAT_PORT% | findstr LISTENING

                    if errorlevel 1 (

                        echo.
                        echo ERROR:
                        echo Tomcat is not listening on port %TOMCAT_PORT%.

                        exit /b 1
                    )

                    echo.
                    echo Tomcat is running successfully.
                '''
            }
        }


        // ============================================================
        // 12. FRONTEND HEALTH CHECK
        // ============================================================

        stage('Frontend Health Check') {

            steps {

                echo '=========================================='
                echo 'FRONTEND HEALTH CHECK'
                echo '=========================================='

                bat '''
                    @echo off

                    echo Frontend URL:
                    echo %FRONTEND_URL%

                    set RETRIES=30

                    :CHECK_FRONTEND

                    echo.
                    echo Checking frontend...
                    echo Attempts remaining: %RETRIES%

                    curl -s -o nul -w "%%{http_code}" "%FRONTEND_URL%" | findstr "200 302"

                    if not errorlevel 1 (

                        echo.
                        echo ==========================================
                        echo FRONTEND IS RUNNING
                        echo ==========================================

                        echo.
                        echo URL:
                        echo %FRONTEND_URL%

                        exit /b 0
                    )

                    set /a RETRIES-=1

                    if %RETRIES% LEQ 0 (

                        echo.
                        echo ==========================================
                        echo FRONTEND HEALTH CHECK FAILED
                        echo ==========================================

                        echo.
                        echo Tomcat port:

                        netstat -ano | findstr :%TOMCAT_PORT%

                        echo.
                        echo Tomcat webapps:

                        dir "%TOMCAT_HOME%\\webapps"

                        echo.
                        echo Tomcat logs:

                        dir "%TOMCAT_HOME%\\logs"

                        exit /b 1
                    )

                    echo Frontend not ready.

                    ping 127.0.0.1 -n 4 >nul

                    goto CHECK_FRONTEND
                '''
            }
        }


        // ============================================================
        // 13. PLAYWRIGHT TESTS
        // ============================================================

        stage('Playwright Tests') {

            steps {

                echo '=========================================='
                echo 'RUNNING PLAYWRIGHT TESTS'
                echo '=========================================='

                bat '''
                    @echo off

                    if not exist "%PLAYWRIGHT_DIR%\\pom.xml" (

                        echo.
                        echo WARNING:
                        echo Playwright Maven project not found.

                        echo.
                        echo Expected:
                        echo %PLAYWRIGHT_DIR%\\pom.xml

                        echo.
                        echo Skipping Playwright tests.

                        exit /b 0
                    )


                    cd /d "%PLAYWRIGHT_DIR%"

                    set "JAVA_HOME=%JAVA_HOME%"
                    set "PATH=%JAVA_HOME%\\bin;%MAVEN_HOME%\\bin;%PATH%"

                    echo.
                    echo Playwright project:
                    echo %PLAYWRIGHT_DIR%

                    echo.
                    echo Frontend:
                    echo %FRONTEND_URL%

                    echo.
                    echo Running Playwright...

                    mvn test

                    set PW_EXIT=%errorlevel%


                    if %PW_EXIT% NEQ 0 (

                        echo.
                        echo ==========================================
                        echo PLAYWRIGHT TESTS FAILED
                        echo ==========================================

                        exit /b %PW_EXIT%
                    )


                    echo.
                    echo ==========================================
                    echo PLAYWRIGHT TESTS PASSED
                    echo ==========================================
                '''
            }
        }
    }


    // ================================================================
    // POST ACTIONS
    // ================================================================

    post {

        success {

            echo '=========================================='
            echo 'MAKER-CHECKER PIPELINE SUCCESSFUL'
            echo '=========================================='

            echo "Backend: ${BACKEND_URL}"
            echo "Frontend: ${FRONTEND_URL}"

            echo '=========================================='
        }


        failure {

            echo '=========================================='
            echo 'MAKER-CHECKER PIPELINE FAILED'
            echo '=========================================='

            echo "Backend log: ${WORKSPACE}\\${PROJECT_DIR}\\backend.log"
            echo "Tomcat logs: ${TOMCAT_HOME}\\logs\\"

            echo '=========================================='
        }
    }
}
