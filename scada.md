---
# Mini Scada System

## Historiador

```C
/*
 =======================================================================
 Name        : Historiador.c
 Author      : Andy Bonilla (19451)
 Version     :
 Copyright   : Digital Electronics 3: Introduction to Embedded Systems
 Description : 
 * 
 =======================================================================

/* =====================================================================
 	 Inclusion de librerias
===================================================================== */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#include <arpa/inet.h>
#include <signal.h>
#include <semaphore.h>
#include <pthread.h>	
#include <sys/stat.h>
#include <sys/types.h>
#include <errno.h>
#include <sys/timerfd.h>
#include <time.h>
#include <sys/wait.h>
/* =====================================================================
	Directivas a ejecutar
===================================================================== */
#define MSG_SIZE 60		// Tamaño (máximo) del mensaje
#define INIT_VALUE	1
#define DataBaseUTRs "DataBaseUTRs.txt"
/* =====================================================================
	Prototipos de funciones
===================================================================== */
void hilo_comunicacion(void *ptr);	//para hilo de comunicacion
void hilo_menu(void *ptr);			//para hilo de evento iot
void error(const char *msg);		//prototipo de funcion de error
char* copyString(char s[]);
/* =====================================================================
	variables globales a implementar
===================================================================== */
sem_t my_semaphore;
//udp
int sockfd, length, n;
socklen_t fromlen;
struct sockaddr_in server;
struct sockaddr_in from;
char buffer[100];
//para datos recibidos
char StringArray[1000][100];		//capaz de registrar 1 hora con 2 UTRs
int puerto,contador=0;
int terminal;
//parte de menu
char input[4];
char mensaje[4];
int continuo=0;
//obtener el tiempo
static int seconds_last=99;
char TimeString[128];
struct timeval current_time;
double delta, total,tiempo_inicial,tiempo_actual;
//

/* =====================================================================
	Interrupts Service Routine (ISR)
===================================================================== */
void ISR()
{
	
}
/* =====================================================================
	main, 2do argumento es el puerto >1024
===================================================================== */
int main(int argc, char *argv[])
{
	continuo=0;
	//----------------------recepcion de argumentos de entrada
	if(argc != 2) 
	{
		fprintf(stderr,"ERROR, no indicó el puerto a usar\n");	
		fprintf(stderr,"Uso: %s num_puerto\n", argv[0]);	
		exit(1);
	}
	
	//udp 
	//----------------------inicializacion de sockets
	sockfd = socket(AF_INET, SOCK_DGRAM, 0); // Creates socket. Connectionless.
	if(sockfd < 0)
		error("Opening socket");;
	//----------------------protocolos de conexion
	length = sizeof(server);			// length of structure
	memset((char *)&server, 0, length); // sets all values to zero.
	server.sin_family = AF_INET;		// symbol constant for Internet domain
	server.sin_addr.s_addr = htonl(INADDR_ANY);	// para recibir de cualquier interfaz de red
	server.sin_port = htons(atoi(argv[1]));	// port number
	//----------------------union de socket a la ip y puerto
	if(bind(sockfd, (struct sockaddr *)&server, length) < 0)
       error("binding");

	fromlen = sizeof(struct sockaddr_in);	// size of structure
	//----------------------inicializacion de valores en semaforo
	sem_init(&my_semaphore, 0, INIT_VALUE);
	//----------------------inicializacion de variables para hilos
	pthread_t var_udp,var_menu;
	//----------------------inicializacion de hilos
	pthread_create(&var_udp, NULL, (void*)&hilo_comunicacion, NULL);
	pthread_create(&var_menu, NULL, (void*)&hilo_menu, NULL);
	//----------------------retorno de hilos
	pthread_join(var_udp,NULL);
	pthread_join(var_menu,NULL);
	//----------------------creacion y union de lineas impares y pares
	printf("Ya se acabaron los hilos\n");
	return 0; 		// cerrar el socket cuando se deja de usar.
	exit(0);
}
/* =====================================================================
	Funciones 
===================================================================== */

/* ==========
	Funcion para hilo de comunicacion
=============*/
void hilo_comunicacion(void *ptr)
{
	
	/* ================================================================
		Loop de comunicacion
	================================================================ */
	while(1)
	{
		/*time_t mytime = time(NULL);
		char * time_str = ctime(&mytime);
		time_str[strlen(time_str)-1] = '\0';
		printf("Current Time : %s\n", time_str);
		sleep(1);*/
		//----------------------buffer se hace 0
		//memset(buffer, 0, MSG_SIZE);
		
		//----------------------recepcion de valores recibidos desde UTRs
		n = recvfrom(sockfd, buffer, 50, 0, (struct sockaddr *)&from, &length);
	    if(n < 0)
	 		error("recvfrom recibido"); 
		//----------------------guarda valores recibidos en base de datos
		sem_wait(&my_semaphore);
		memcpy(StringArray[contador],buffer,strlen(buffer));
		sem_post(&my_semaphore);
		
		//----------------------ver si se despliegue desde menu
		if (continuo==0)
		{
			contador++;
		}
		else if(continuo==1)
		{
			puts("U E Tiempo                   Sw Bt Led V \n");
			printf("%s\n",StringArray[contador]);
			contador++;
			fflush(stdout);
		}
		//----------------------mensaje de recibido
		n = sendto(sockfd, mensaje, MSG_SIZE, 0, (struct sockaddr *)&from, fromlen);
		if(n < 0)
	 		error("sendto");
		if (contador>999)
			contador=0;
	} 	
	

	pthread_exit(0);
}

/* ==========
	Funcion para hilo de evento IoT
=============*/
void hilo_menu(void *ptr)
{
	FILE *datos = fopen(DataBaseUTRs, "w");
	fputs("U E Tiempo   Sw Bot LED V\n", datos);
	/* =======================================================================
		Loop de eventos IoT
	========================================================================= */
	while(1)
	{
		//----------------------menu principal
		printf("--------------------------------------\n");
		printf("--------------------------------------\n");
		printf("  ¡BIENVENIDO AL HISOTRIADOR!\n");
		printf("  Ingrese el menu que desea desplegar\n");
		printf("  Despliegue continuo: con\n");
		printf("  Encender LEDs en UTRs: uon\n");
		printf("  Apagar LEDs en UTRs: off\n");
		printf("  Desplegar todo el historial: ver\n");
		printf("--------------------------------------\n");
		printf("--------------------------------------\n");
		scanf("%9s", input);
		//----------------------despliegue de datos continuo
		if (strncmp(input, "con",3)==0)		
		{
			continuo=1;
			printf("  DESPLIEGUE CONTINUO\n");
			puts("U E Tiempo   Sw Bt Led V n");
			//continuo=1;
		}
		else 
			continuo=0;
		//----------------------control de leds en UTRs
		if (strcmp(input, "uon")==0)	
		{
			continuo=0;
			printf("--------------------------------------\n");
			printf("  Seleccione los LEDs que desea encender\n");	
			printf("  UTR1 LED1: L11\n");
			printf("  UTR1 LED2: L12\n");
			printf("  UTR1 LED1: L21\n");
			printf("  UTR2 LED2: L22\n");
			printf("  Todas: all\n");
			printf("--------------------------------------\n");
			scanf("%9s", input);
			if (strcmp(input, "L11")==0)
			{
				sprintf(mensaje,"111\n");
				printf("Haz prendido el Led1 UTR1\n");
			}
			else if (strcmp(input, "L12")==0)
			{
				sprintf(mensaje,"112\n");
				printf("Haz prendido el Led2 UTR1\n");
			}
			else if (strcmp(input, "L21")==0)
			{
				sprintf(mensaje,"121\n");
				printf("Haz prendido el Led1 UTR2\n");
			}
			else if (strcmp(input, "L22")==0)
			{
				sprintf(mensaje,"122\n");
				printf("Haz prendido el Led2 UTR2\n");
			}
			else if (strcmp(input, "all")==0)
			{
				sprintf(mensaje,"all\n");
				printf("Haz prendido todos los LEDs\n");
			}
		}
		//----------------------apagar UTRS
		if (strcmp(input, "off")==0)			
		{
			continuo=0;
			printf("--------------------------------------\n");
			printf("Seleccione los LEDs que desea apagar\n");	
			printf("  UTR1 LED1: L11\n");
			printf("  UTR1 LED2: L12\n");
			printf("  UTR1 LED1: L21\n");
			printf("  UTR2 LED2: L22\n");
			printf("  Todas: all\n");
			printf("--------------------------------------\n");
			scanf("%9s", input);
			if (strcmp(input, "L11")==0)
			{
				sprintf(mensaje,"011\n");
				printf("Haz apagado el Led1 UTR1\n");
			}
			else if (strcmp(input, "L12")==0)
			{
				sprintf(mensaje,"012\n");
				printf("Haz apagado el Led2 UTR1\n");
			}
			else if (strcmp(input, "L21")==0)
			{
				sprintf(mensaje,"021\n");
				printf("Haz apagado el Led1 UTR2\n");
			}
			else if (strcmp(input, "L22")==0)
			{
				sprintf(mensaje,"022\n");
				printf("Haz apagado el Led2 UTR2\n");
			}
			if (strcmp(input, "all")==0)
			{
				sprintf(mensaje,"000\n");
				printf("Haz apagado el Led2 UTR2\n");
			}
		}
		//----------------------despliegue de datos almacenados
		if (strncmp(input, "ver",3)==0)		
		{
			printf("Desplegando historial\n");
			//fputs("U E Tiempo   Sw Bot LED V\n", datos);
			for(int i = 0; i < 1000 ; i++)
			{
				printf("%s\n",StringArray[i]); 
				printf("\n");
				//fputs(StringArray[i], datos);
				fputs(StringArray[i], datos);
			}
			fclose(datos); //se cierra el nuevo archivo
			
		}

	}
	//----------------------se sale del hilo
	pthread_exit(0);
}


/* ==========
	Funcion para desplegar mensajes de error
=============*/
void error(const char *msg)
{
    perror(msg);	// escribe al standard error (la terminal, por defecto)
    exit(1);
}


char* copyString(char s[])
{
    char* s2;
    s2 = (char*)malloc(20);
 
    strcpy(s2, s);
    return (char*)s2;
}
```


## RTU (Remote Terminal Unit)
This C program, designed for Raspberry Pi, implements a multithreaded sampling and communication system. It integrates an SPI-connected ADC (MCP3002) for analog-to-digital conversion, digital input/output controls for switches, buttons, and LEDs, and uses UDP sockets for network communication. The system captures ADC readings, processes voltage values, and detects out-of-range conditions to trigger events. It also handles user interactions through interrupts (switches and buttons), synchronizes access with semaphores, and logs events with timestamps. Multiple threads are responsible for sampling the ADC, sending collected data via UDP, and receiving remote commands to control outputs (like LEDs and buzzer). Overall, the program provides a real-time framework for monitoring, event detection, and remote IoT-style communication.

```C
/*
 =======================================================================
 Name        : Muestreo.c
 Author      : Andy Bonilla (19451) 
 Copyright   : Digital Electronics 3: Introduction to Embedded Systems
 Description : 
 * 
 =======================================================================

/* =====================================================================
 	 Library inclusion
===================================================================== */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <unistd.h>
#include <stdint.h>			//integers like uint8_t and uint16_t
#include <wiringPi.h>		//wiringPi for gpio
#include <wiringPiSPI.h>	//wiringPi for SPI connection
#include <pthread.h>		//for multi-threading
#include <string.h>			//string handling
#include <sys/types.h>
#include <sys/socket.h>		//use of sockets
#include <netinet/in.h>		//network access
#include <netdb.h>			//network access
#include <arpa/inet.h>		//network access
#include <sys/time.h>
#include <time.h>
#include <sys/timerfd.h>
#include <semaphore.h>
* =====================================================================
	Directives to execute
===================================================================== */
//for ADC IC connection
#define SPI_CHANNEL	      0	// SPI channel of Raspberry Pi, 0 or 1
#define SPI_SPEED 	1500000	// SPI communication speed (clock, in HZ)
#define ADC_CHANNEL       0	// A/D channel of MCP3002 to use, 0 or 1
//for message size
//via tcp
#define MSG_SIZE 100		// Maximum message size
//for semaphore use
#define INIT_VALUE	1		// Initial value of semaphore
//for timer
#define milis_nanos 1000000
/* =====================================================================
	Function prototypes
===================================================================== */
uint16_t get_ADC(int channel);			//for adc
void funcion_adc(void *ptr);			//for sampling with adc
void funcion_envio(void *ptr);			//for sending data via udp
void funcion_recibir(void *ptr);		//for receiving data via udp
void funcion_eventos(int ev);
void error(const char *msg);			//errors in tcp communication
void espera(int tiempo);
/* =====================================================================
	Global variables
===================================================================== */
//for semaphore
sem_t my_semaphore;
//for debouncing digital inputs
int antirrebote1,antirrebote2,antirrebote3=0,antirrebote4=0;
int inter1,inter2,bot1,bot2;	//for debouncing
int led1,led2;
//for sending messages
char datos_udp[50];								//for tcp messages
int eventos=0;
//for ADC voltage conversion
uint16_t ADCvalue;
uint16_t conversion;
float voltaje;
uint16_t i;
//for socket communication
int sockfd, n;
unsigned int length;
struct sockaddr_in server, from;
struct hostent *hp;
char buffer[MSG_SIZE];
//to get time
static int seconds_last=99;
char TimeString[128];
struct timeval current_time;
double delta,total,tiempo_inicial,tiempo_actual;
//event array
char recopilacion[1000000][50];
char datos_eventos[100];
int contador=0;
//to check if there was update
int overshoot=0,undershoot=0;
int bandera;
/* =====================================================================
	Interrupts Service Routine (ISR)
===================================================================== */
void ISR()
{
	//----------------------for switch 1 state change
	if (digitalRead(12)==1)
	{
		inter1=1;
		usleep(250000);
		funcion_eventos(1);
	}
	else if (digitalRead(12)==0)
	{
		usleep(250000);
		inter1=0;
	}
	//----------------------for switch 2 state change
	if (digitalRead(16)==1 )
	{
		inter2=1;
		usleep(250000);
		funcion_eventos(2);
	}
	else if (digitalRead(12)==0)
	{
		usleep(250000);
		inter2=0;
	}
	//----------------------for button 1 state change
	if (digitalRead(20)==0 )
	{
		bot1=1;
		usleep(250000);
		funcion_eventos(3);
	}
	//----------------------for button 2 state change
	if (digitalRead(18)==0 )
	{
		bot2=1;
		usleep(250000);
		funcion_eventos(4);
	}
	//----------------------for IoT event 1
	if (digitalRead(22)==1 )
	{
		led1=1;
		usleep(250000);
		funcion_eventos(7);
		digitalWrite(23,1);
	}
	else if  (digitalRead(22)==0)
	{
		usleep(250000);
		digitalWrite(23,0);
		led1=0;
	}
	//----------------------for IoT event 2
	if (digitalRead(27)==1 )
	{
		led2=1;
		usleep(250000);
		funcion_eventos(8);
		digitalWrite(24,1);
	}
	else if (digitalRead(27)==0)
	{
		usleep(250000);
		led2=0;
		digitalWrite(24,0);
	}
}

/* =====================================================================
	main
===================================================================== */
int main(int argc, char *argv[])
{
   //----------------------input arguments reception
    if(argc != 3)	
    {
		fprintf(stderr,"ERROR, Uso correcto: %s IP_servidor num_puerto\n", argv[0]);
		exit(0);
    }
    //----------------------socket creation
    sockfd = socket(AF_INET, SOCK_DGRAM, 0); 
	if(sockfd < 0)
		error("socket");
	//----------------------server ip fetching
	server.sin_family = AF_INET;	
	hp = gethostbyname(argv[1]);	
	if(hp == 0)
		error("Unknown host");
    //----------------------port and address creation
    memcpy((char *)&server.sin_addr, (char *)hp->h_addr, hp->h_length);
	server.sin_port = htons(atoi(argv[2]));	
	length = sizeof(struct sockaddr_in);	
	//---------------------- pin configuration
	wiringPiSetupGpio();
	pinMode(12,INPUT);						// mode switch 1
	pinMode(16,INPUT);						// mode switch 2
	pinMode(20,INPUT);						// mode button 1
	pinMode(18,INPUT);						// mode button 2
	pinMode(23,OUTPUT);						// mode led1
	pinMode(24,OUTPUT);						// mode led2
	pinMode(25,OUTPUT);						// mode buzzer
	pinMode(22,OUTPUT);						// mode IoT1
	pinMode(27,OUTPUT);						// mode IoT2
	pullUpDnControl(12,PUD_UP);				// pull up switch 1
	pullUpDnControl(16,PUD_UP);				// pull up switch 2
	pullUpDnControl(20,PUD_UP);				// pull up button 1
	pullUpDnControl(18,PUD_UP);				// pull up button 2
	pullUpDnControl(22,PUD_UP);				// pull up IOT 1
	pullUpDnControl(27,PUD_UP);				// pull up IOT 2
	wiringPiISR(12, INT_EDGE_BOTH,ISR);	// interrupt switch 1
	wiringPiISR(16, INT_EDGE_BOTH,ISR);	// interrupt switch 2
	wiringPiISR(20, INT_EDGE_RISING,ISR);	// interrupt button 1
	wiringPiISR(18, INT_EDGE_RISING,ISR);	// interrupt button 2
	wiringPiISR(22, INT_EDGE_BOTH,ISR);	// interrupt IoT 1
	wiringPiISR(27, INT_EDGE_BOTH,ISR);	// interrupt IoT 2
	//----------------------start time function
	gettimeofday(&current_time,NULL);
	tiempo_inicial=current_time.tv_sec + current_time.tv_usec/1000000.0;
	//----------------------ADC configuration
	if(wiringPiSPISetup(SPI_CHANNEL, SPI_SPEED) < 0)
	{
		printf("wiringPiSPISetup falló.\n");
		return(-1);
	}
	//----------------------semaphore init
	sem_init(&my_semaphore, 0, INIT_VALUE);
	//----------------------thread variables init 
	pthread_t var_adc,var_switch,var_envio,var_recibir;	
	//----------------------thread init
	pthread_create(&var_adc, NULL, (void*)&funcion_adc, NULL);
	pthread_create(&var_envio, NULL, (void*)&funcion_envio, NULL);
	pthread_create(&var_recibir, NULL, (void*)&funcion_recibir, NULL);
	//----------------------join threads
	pthread_join(var_adc,NULL);
	pthread_join(var_envio,NULL);
	pthread_join(var_recibir,NULL);
	printf("Ya se terminaron los hilos\n");
	return 0;
	exit(0);
}
/* =====================================================================
	Functions for threads
===================================================================== */
/* ==========
	Get ADC values
=============*/
uint16_t get_ADC(int ADC_chan)
{
	//----------------------bytes a usar 
	uint8_t spiData[2];	
	uint16_t resultado;	
	//----------------------validacion de canal
	if((ADC_chan < 0) || (ADC_chan > 1))
		ADC_chan = 0;
	//----------------------byte de configuración
	spiData[0] = 0b01101000 | (ADC_chan << 4);  
	spiData[1] = 0;									
	//----------------------escritura/lectura de SPI.
	wiringPiSPIDataRW(SPI_CHANNEL, spiData, 2);	// 2 bytes
	resultado = (spiData[0] << 8) | spiData[1];
	return(resultado);
}
/* ==========
	ADC sampling
=============*/
void funcion_adc(void *ptr)
{
	//----------------------configuracion del timer
	int timer_fd1 = timerfd_create(CLOCK_MONOTONIC, 0);
	struct itimerspec itval;
	itval.it_interval.tv_sec = 0;
	itval.it_interval.tv_nsec = 100000000;		//100ms
	itval.it_value.tv_sec = 0;
	itval.it_value.tv_nsec = 1000000;			//1ms
	if(	timerfd_settime(timer_fd1, 0, &itval, NULL) == 1)
	{
		perror("Error al hacer el timer.\n");
		exit(0);
	}
	//----------------------ciclo de ejecucion
	while(1)
	{
		//----------------------conversion de adc
		ADCvalue = get_ADC(ADC_CHANNEL);
		fflush(stdout);
		voltaje=(ADCvalue*3.3)/1203.0;
		//----------------------verificacion dentro de rango 
		if (voltaje>0.5 && voltaje<2.5)
		{
			if (bandera==0)
			{
				funcion_eventos(6);
				bandera=1;
			}
		}
		//----------------------verificacion fuera de rango
		if (voltaje>2.5 || voltaje<0.5)
		{
			digitalWrite(25,1);
			if (bandera==1)
			{
				
				funcion_eventos(5);
				bandera=0;
			}
		}
		else
			digitalWrite(25,0);
		//----------------------configuracion y arranque de timer
		espera(timer_fd1);
	}
	//----------------------salida del hilo
	pthread_exit(0);
}

/* ==========
	Sending thread data  
=============*/
void funcion_envio(void *ptr)
{
	puts("U E Tiempo                   Sw Bt Led V \n");
	while(1)
	{			
		//inter1=0,inter2=0,bot1=0,bot2=0;
		funcion_eventos(0);	
		//----------------------si en caso hubo algun evento
		for(int j=0;j<contador+1;j++)
		{
			
			//----------------------mandar datos via udp
			n = sendto(sockfd, recopilacion[j], strlen(recopilacion[j]), 0, (const struct sockaddr *)&server, length);
			if(n < 0)
				error("Sendto");
			
			puts(recopilacion[j]);
			fflush(stdout);
			
		}
		//funcion_eventos(0);
		contador=0;
		sleep(2);
	}
	//----------------------salida del hilo
	pthread_exit(0);
}

/* ==========
	Receiving thread data  
=============*/
void funcion_recibir(void *ptr)
{
	while(1)
	{	
		//----------------------se recibe del server
		n = recvfrom(sockfd, buffer, MSG_SIZE, 0, (struct sockaddr *)&from, &length);
		if(n < 0)
			error("recvfrom");
		//printf(buffer);
		//----------------------para prender/apagar led1
		if (strcmp("111\n",buffer)==0)
		{
			digitalWrite(23,1);
			led1=1;
		}
		else if (strcmp("011\n",buffer)==0)
		{
			digitalWrite(23,0);
			led1=0;
		}
		//----------------------para prender/apagar led2
		if (strcmp("112\n",buffer)==0)
		{
			digitalWrite(24,1);
			led2=1;
		}	
		else if (strcmp("012\n",buffer)==0)
		{
			digitalWrite(24,0);
			led2=1;
		}
		//----------------------para prender/apagar led2
		if (strcmp("all\n",buffer)==0)
		{
			digitalWrite(23,1);
			digitalWrite(24,1);
			led1=1;
			led2=1;
		}
		if (strcmp("000\n",buffer)==0)
		{
			digitalWrite(23,0);
			digitalWrite(24,0);
			led1=0;
			led2=0;
		}		
	}
}

/* ==========
	Events thread
=============*/
void funcion_eventos(int ev)
{
	//memset(datos_eventos, 0, MSG_SIZE);	// sets all values to zero
	//----------------------obtener el tiempo actual
	time_t mytime=time(NULL);
	char * time_str=ctime(&mytime);
	time_str[strlen(time_str)-1]='\0';
	//----------------------escritura de arreglo
	memset(datos_eventos, 0, MSG_SIZE);	// sets all values to zero
	sprintf(datos_eventos,"1 %d %s %d%d %d%d %d%d %f\n",ev,time_str,inter1,inter2,bot1,bot2,led1,led2,voltaje);
	fflush(stdout);
	//----------------------se copia a estructura
	strcpy(recopilacion[contador],datos_eventos); 	//copia buffer en array
	fflush(stdout); 		
	contador++;		
}
/* ==========
	Comms error
=============*/
void error(const char *msg)
{
	perror(msg);
    exit(0);
}
/* ==========
	Wait time
=============*/
void espera(int tiempo)
{
	uint64_t periodos=0;

	if(read(tiempo,&periodos,sizeof(periodos))==-1)
	{
		perror("Error al leer el timer");
		exit(1);
	}
	if (periodos>1)
	{
		puts("Se pasó el tiempo");
		exit(1);
	}
}


```
