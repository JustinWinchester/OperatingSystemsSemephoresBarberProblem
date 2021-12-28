// compiling line: cc -o executable source_code -lpthread
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <semaphore.h>

#define MAX 30

void * customer(void*);
void* barber(void*);
void randwait(int n);

sem_t waitingroom, barberchair, barberwakeup, seatbelt;

int alldone = 0;
int who;

int main(int argc, char* argv[])
{
	pthread_t bartid, customers[MAX];
	long randseed;
	int i, num_chairs, num_customers;
	int number[MAX];

	if(argc != 4)

	{
		printf("\nUse: Sleepbarber x y z \n");
		printf("where x = # of customers, y = # of chairs, ");
		printf("and z = random number seed.\n\n");
		exit(-1);
	}

	//initialize variables
	num_customers = atoi(argv[1]);
	num_chairs = atoi(argv[2]);
	randseed = atoi(argv[3]);

	if(num_customers > MAX)
	{
		printf("\nToo many customers.");
		num_customers = MAX;
	}
	
	srand48(randseed);

	for(i =0; i<num_customers; i++)
	{
		number[i] = i;
	}

	sem_init(&waitingroom, 0, 1);//num_chairs);
	sem_init(&barberchair, 0, 1);
	sem_init(&barberwakeup, 0, 0);
	sem_init(&seatbelt, 0, 0);

	pthread_create(&bartid, NULL, barber, NULL);

	for(i=0; i<num_customers; i++)
	{
		pthread_create(&customers[i], NULL, customer, 
				(void*)&number[i]);
	}

	for(i=0; i<num_customers; i++)
	{
		pthread_join(customers[i], NULL);
	}

	alldone = 1;

	sem_post(&barberwakeup);

	pthread_join(bartid, NULL);

}

void* customer(void* n)
{
	int num = *(int*) n;

	printf("\nCustomer %d leaving for barber shop.\n", num);
	randwait(3);

	printf("\nCustomer %d arrived at barber shop\n", num);

	sem_wait(&waitingroom);

	printf("\nCustomer %d enter waiting room.\n", num);

	sem_wait(&barberchair); //waiting for barber chair

	sem_post(&waitingroom); //signal a chair available in WR

	printf("\nCustomer %d waking the barber..\n", num);

	sem_post(&barberwakeup);
	who = num;

	sem_wait(&seatbelt); //let barber cut his hair

	sem_post(&barberchair); //make barberchair available again

	printf("\nCustomer %d leaving barber shop ....... \n", num);
}

void* barber(void* useless)
{
	while(!alldone)
	{
		printf("\nBarber is sleeping ......\n");
		sem_wait(&barberwakeup);

		if(!alldone)
		{
		printf("\nthe barber is cuting %d hair.\n", who);
		randwait(3);

		printf("\nThe barber finished cutting %d hair. \n", who);
		sem_post(&seatbelt);//release the customer
		}
		else
		{
			printf("\nBarber is going home ......\n");
		}
	}//end of while loop
}

void randwait(int n)
{
	int len;

	len = (int) ((drand48()*n)+1);
	sleep(len);
}
	

	