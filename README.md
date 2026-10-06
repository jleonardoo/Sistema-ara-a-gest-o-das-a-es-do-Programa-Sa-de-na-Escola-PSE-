# Sistema para a gestão das açeõs do Programa Saúde na Escola (PSE)
Projetar e planejar um protótipo de sistema computacional para a gestão integrada das ações do Programa Saúde na Escola (PSE) 


#include <stdio.h>

main(){
	int conta;
	printf("\n=====================================\n");
	printf("             SISTEMA DO PSE              ");
	printf("\n=====================================\n");
	
	printf("\n01 - criar conta");
	printf("\n02 - entrar ");
	
	scanf("%i", &conta);
	
	switch(conta){
		case 1:
			printf("deu");
		break;
		case 2:
			printf("foi");
	    break;
		default:
			printf("esse tbm");
	}

}
