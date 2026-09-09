@echo off
setlocal EnableDelayedExpansion

REM ============================================================
REM AUDITORIA BASICA DE RED LOCAL - WINDOWS
REM Utiliza solamente comandos nativos de Windows.
REM ============================================================

title Auditoria Basica de Red Local

REM ------------------------------------------------------------
REM 1. CREAR NOMBRE DEL REPORTE
REM ------------------------------------------------------------

for /f "tokens=1-3 delims=/ " %%a in ("%date%") do (
    set FECHA=%%c-%%b-%%a
)

for /f "tokens=1-2 delims=: " %%a in ("%time%") do (
    set HORA=%%a%%b
)

set HORA=!HORA: =0!

set REPORTE=reporte_red_!FECHA!_!HORA!.txt

REM ------------------------------------------------------------
REM 2. ENCABEZADO DEL REPORTE
REM ------------------------------------------------------------

echo ============================================================ > "%REPORTE%"
echo           AUDITORIA BASICA DE RED LOCAL                    >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo Fecha: %date%                                               >> "%REPORTE%"
echo Hora:  %time%                                               >> "%REPORTE%"
echo Equipo: %COMPUTERNAME%                                     >> "%REPORTE%"
echo Usuario: %USERNAME%                                        >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo.                                                            >> "%REPORTE%"

REM ------------------------------------------------------------
REM 3. INFORMACION DE RED LOCAL
REM ------------------------------------------------------------

echo [1] INFORMACION DE RED LOCAL
echo.

echo ============================================================ >> "%REPORTE%"
echo [1] INFORMACION DE RED LOCAL                               >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

ipconfig /all >> "%REPORTE%"

REM ------------------------------------------------------------
REM 4. OBTENER IP LOCAL Y GATEWAY
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ------------------------------------------------------------ >> "%REPORTE%"
echo RESUMEN DE CONFIGURACION IP                                >> "%REPORTE%"
echo ------------------------------------------------------------ >> "%REPORTE%"

for /f "tokens=2 delims=:" %%A in ('ipconfig ^| findstr /R /C:"IPv4.*"') do (
    set IP=%%A
    set IP=!IP: =!
    goto IP_ENCONTRADA
)

:IP_ENCONTRADA

echo IP local detectada: !IP! >> "%REPORTE%"

for /f "tokens=2 delims=:" %%A in ('ipconfig ^| findstr /R /C:"Mascara de subred.*"') do (
    set MASCARA=%%A
    set MASCARA=!MASCARA: =!
    goto MASCARA_ENCONTRADA
)

:MASCARA_ENCONTRADA

echo Mascara de subred: !MASCARA! >> "%REPORTE%"

for /f "tokens=2 delims=:" %%A in ('ipconfig ^| findstr /R /C:"Puerta de enlace predeterminada.*"') do (
    set GATEWAY=%%A
    set GATEWAY=!GATEWAY: =!
    if not "!GATEWAY!"=="" goto GATEWAY_ENCONTRADO
)

:GATEWAY_ENCONTRADO

echo Puerta de enlace: !GATEWAY! >> "%REPORTE%"

REM ------------------------------------------------------------
REM 5. SERVIDORES DNS
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo Servidores DNS: >> "%REPORTE%"

ipconfig /all | findstr /I /C:"Servidores DNS" /C:"DNS Servers" >> "%REPORTE%"

REM ------------------------------------------------------------
REM 6. DETERMINAR SUBRED
REM ------------------------------------------------------------

echo.
echo [2] DETERMINANDO SUBRED...

REM Se toma la IP IPv4 detectada y se reemplaza el ultimo octeto
REM por 1-254.

for /f "tokens=1-4 delims=." %%a in ("!IP!") do (
    set RED=%%a.%%b.%%c
)

echo Subred detectada: !RED!.0/24 >> "%REPORTE%"

echo. >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo [2] ESCANEO DE DISPOSITIVOS ACTIVOS                         >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo Subred analizada: !RED!.0/24 >> "%REPORTE%"
echo. >> "%REPORTE%"

echo Escaneando !RED!.1 hasta !RED!.254...
echo.

REM ------------------------------------------------------------
REM 7. PING DEL RANGO .1 - .254
REM ------------------------------------------------------------

for /L %%i in (1,1,254) do (

    ping -n 1 -w 150 !RED!.%%i >nul

    if not errorlevel 1 (
        echo [ACTIVO] !RED!.%%i
        echo [ACTIVO] !RED!.%%i >> "%REPORTE%"
    )

)

REM ------------------------------------------------------------
REM 8. TABLA ARP
REM ------------------------------------------------------------

echo.
echo [3] TABLA ARP
echo.

echo ============================================================ >> "%REPORTE%"
echo [3] TABLA ARP - IP Y MAC                               >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

arp -a >> "%REPORTE%"

REM ------------------------------------------------------------
REM 9. TABLA ARP LIMPIA
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ------------------------------------------------------------ >> "%REPORTE%"
echo IP Y MAC DETECTADAS                                        >> "%REPORTE%"
echo ------------------------------------------------------------ >> "%REPORTE%"

arp -a | findstr /R /C:"[0-9][0-9]*\.[0-9][0-9]*\.[0-9][0-9]*\.[0-9][0-9]*" >> "%REPORTE%"

REM ------------------------------------------------------------
REM 10. RECURSOS COMPARTIDOS
REM ------------------------------------------------------------

echo.
echo [4] RECURSOS COMPARTIDOS
echo.

echo ============================================================ >> "%REPORTE%"
echo [4] RECURSOS COMPARTIDOS                                   >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

echo Recursos compartidos del equipo: >> "%REPORTE%"
net share >> "%REPORTE%"

REM ------------------------------------------------------------
REM 11. EQUIPOS VISIBLES EN LA RED
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ------------------------------------------------------------ >> "%REPORTE%"
echo EQUIPOS VISIBLES EN LA RED                                 >> "%REPORTE%"
echo ------------------------------------------------------------ >> "%REPORTE%"

net view >> "%REPORTE%" 2>&1

REM ------------------------------------------------------------
REM 12. CONEXIONES TCP ACTIVAS
REM ------------------------------------------------------------

echo.
echo [5] CONEXIONES TCP ACTIVAS
echo.

echo ============================================================ >> "%REPORTE%"
echo [5] CONEXIONES TCP ACTIVAS                                  >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

netstat -ano -p tcp >> "%REPORTE%"

REM ------------------------------------------------------------
REM 13. PUERTOS UDP
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo [6] PUERTOS UDP                                             >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

netstat -ano -p udp >> "%REPORTE%"

REM ------------------------------------------------------------
REM 14. PROCESOS EN EJECUCION
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo [7] PROCESOS EN EJECUCION                                   >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

tasklist >> "%REPORTE%"

REM ------------------------------------------------------------
REM 15. PUERTOS ESCUCHANDO
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo [8] PUERTOS TCP EN ESCUCHA                                 >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

netstat -ano -p tcp | findstr LISTENING >> "%REPORTE%"

REM ------------------------------------------------------------
REM 16. INFORMACION DE RUTAS
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo [9] TABLA DE RUTAS                                         >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

route print >> "%REPORTE%"

REM ------------------------------------------------------------
REM 17. CONFIGURACION DE FIREWALL
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo [10] ESTADO DEL FIREWALL DE WINDOWS                         >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

netsh advfirewall show allprofiles >> "%REPORTE%"

REM ------------------------------------------------------------
REM 18. FINAL DEL REPORTE
REM ------------------------------------------------------------

echo. >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"
echo FIN DE LA AUDITORIA                                        >> "%REPORTE%"
echo ============================================================ >> "%REPORTE%"

echo.
echo ============================================================
echo AUDITORIA FINALIZADA
echo ============================================================
echo.
echo Reporte generado:
echo %CD%\%REPORTE%
echo.
echo Abriendo reporte...
echo.

start "" "%REPORTE%"

endlocal
pause