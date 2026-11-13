# Sistema para a gestão das açeõs do Programa Saúde na Escola (PSE)
Projetar e planejar um protótipo de sistema computacional para a gestão integrada das ações do Programa Saúde na Escola (PSE).

#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <locale.h>
 
#define MAX_CONTAS 100
#define MAX_ESCOLAS 100
#define MAX_ACOES 500
 
typedef struct {
    char usuario[50];
    char senha[50];
    int tipo;
} Conta;
 
Conta contas[MAX_CONTAS];
int totalcontas = 0;
 
typedef struct {
    int id;
    char nome[60];
    char municipio[40];
    int ativo;
} Escola;
 
typedef struct {
    int id;
    int idEscola;
    char tema[40];
    char descricao[100];
    char data[11];
    int status;
    int participantes;
    int ativo;
} Acao;
 
Escola escolas[MAX_ESCOLAS];
Acao acoes[MAX_ACOES];
int totalEscolas = 0;
int totalAcoes = 0;
 
void criarconta(Conta *novaconta) {
 
    printf("\n=====================================\n");
    printf("              CRIAR CONTA\n");
    printf("=====================================\n");
 
    if (totalcontas >= MAX_CONTAS) {
        printf("Limite de contas atingido!\n");
        return;
    }
 
    printf("Digite um nome de usuário:\n");
    scanf("%49s", novaconta->usuario);
 
    printf("Digite uma senha:\n");
    scanf("%49s", novaconta->senha);
 
    printf("\nTipo de conta:\n");
    printf("1 - Administrador\n");
    printf("2 - Profissional da saúde\n");
    printf("3 - Profissional da educação\n");
    printf("Escolha: ");
    scanf("%d", &novaconta->tipo);
 
    if (novaconta->tipo < 1 || novaconta->tipo > 3) {
        printf("Tipo de conta invalido!\n");
        return;
    }
 
    contas[totalcontas] = *novaconta;
    totalcontas++;
 
    printf("\nConta criada com sucesso!\n");
}
 
int login() {
    char usuario[50];
    char senha[50];
    int i;
 
    if (totalcontas == 0) {
        printf("\nNenhuma conta cadastrada!\n");
        return 0;
    }
 
    printf("\n=====================================\n");
    printf("                 LOGIN\n");
    printf("=====================================\n");
 
    printf("Digite seu nome de usuário: ");
    scanf("%49s", usuario);
 
    printf("Digite sua senha: ");
    scanf("%49s", senha);
 
    for (i = 0; i < totalcontas; i++) {
 
        if (strcmp(contas[i].usuario, usuario) == 0 &&
            strcmp(contas[i].senha, senha) == 0) {
 
            printf("\nLogin realizado com sucesso!\n");
 
            return contas[i].tipo;
        }
    }
 
    printf("\nUsuário ou senha incorretos!\n");
 
    return 0;
}

void limparBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) { }
}
 
int lerTexto(const char *msg, char *dest, int tam) {
    printf("%s", msg);
    if (fgets(dest, tam, stdin) == NULL) {
        dest[0] = '\0';
        return 0;
    }
    size_t len = strlen(dest);
    if (len > 0 && dest[len - 1] == '\n') {
        dest[len - 1] = '\0';
    } else {
        limparBuffer();
    }
    return (int)strlen(dest);
}
 
int lerInteiro(const char *msg, int min, int max) {
    char buf[32];
    int valor;
    char extra;
    while (1) {
        lerTexto(msg, buf, sizeof(buf));
        if (sscanf(buf, "%d %c", &valor, &extra) == 1 && valor >= min && valor <= max)
            return valor;
        printf("  Valor inválido! Digite um número entre %d e %d.\n", min, max);
    }
}
 
void pausar() {
    char buf[8];
    lerTexto("\nPressione ENTER para continuar...", buf, sizeof(buf));
}
 
const char *nomeStatus(int s) {
    switch (s) {
        case 0: return "Planejada";
        case 1: return "Em andamento";
        case 2: return "Concluída";
        case 3: return "Cancelada";
        default: return "?";
    }
}
 
Escola *buscarEscola(int id) {
    if (id < 1 || id > totalEscolas || !escolas[id - 1].ativo) return NULL;
    return &escolas[id - 1];
}
 
Acao *buscarAcao(int id) {
    if (id < 1 || id > totalAcoes || !acoes[id - 1].ativo) return NULL;
    return &acoes[id - 1];
}
 
 // crud escola
 
void listarEscolas() {
    int i, achou = 0;
    printf("\n--- ESCOLAS ---\n");
    for (i = 0; i < totalEscolas; i++) {
        if (!escolas[i].ativo) continue;
        printf("[%d] %-35s %s\n", escolas[i].id, escolas[i].nome, escolas[i].municipio);
        achou = 1;
    }
    if (!achou) printf("Nenhuma escola cadastrada.\n");
}
 
void cadastrarEscola() {
    Escola e;
    if (totalEscolas >= MAX_ESCOLAS) {
        printf("Limite de escolas atingido.\n");
        return;
    }
    memset(&e, 0, sizeof(e));
    printf("\n--- NOVA ESCOLA ---\n");
    if (!lerTexto("Nome da escola: ", e.nome, sizeof(e.nome))) { printf("Nome obrigatório.\n"); return; }
    if (!lerTexto("Município: ", e.municipio, sizeof(e.municipio))) { printf("Município obrigatório.\n"); return; }
    e.id = totalEscolas + 1;
    e.ativo = 1;
    escolas[totalEscolas] = e;
    totalEscolas++;
    printf("Escola cadastrada com ID %d.\n", e.id);
}
 
void atualizarEscola() {
    int id;
    Escola *e;
    char msg[100], novo[60];
 
    listarEscolas();
    id = lerInteiro("\nID da escola a editar (0 = cancelar): ", 0, MAX_ESCOLAS);
    if (id == 0) return;
    e = buscarEscola(id);
    if (!e) { printf("Escola não encontrada.\n"); return; }
 
    printf("(Deixe em branco para manter o valor atual)\n");
    sprintf(msg, "Nome [%s]: ", e->nome);
    if (lerTexto(msg, novo, sizeof(novo))) strcpy(e->nome, novo);
    sprintf(msg, "Município [%s]: ", e->municipio);
    if (lerTexto(msg, novo, 40)) strcpy(e->municipio, novo);
    printf("Escola atualizada.\n");
}
 
void excluirEscola() {
    int id, i;
    Escola *e;
 
    listarEscolas();
    id = lerInteiro("\nID da escola a excluir (0 = cancelar): ", 0, MAX_ESCOLAS);
    if (id == 0) return;
    e = buscarEscola(id);
    if (!e) { printf("Escola não encontrada.\n"); return; }
    for (i = 0; i < totalAcoes; i++) {
        if (acoes[i].ativo && acoes[i].idEscola == id) {
            printf("Não é possível excluir: existem ações vinculadas a esta escola.\n");
            return;
        }
    }
    e->ativo = 0;
    printf("Escola excluída.\n");
}
 
void menuEscolas(int tipo) {
    int op;
    do {
        printf("\n=== ESCOLAS ===\n");
        printf("1 - Listar\n");
        if (tipo == 1) {
            printf("2 - Cadastrar\n");
            printf("3 - Atualizar\n");
            printf("4 - Excluir\n");
        }
        printf("0 - Voltar\n");
        op = lerInteiro("Opção: ", 0, 4);
        if (op == 1) listarEscolas();
        else if (op >= 2 && tipo != 1) printf("Acesso negado: apenas administradores.\n");
        else if (op == 2) cadastrarEscola();
        else if (op == 3) atualizarEscola();
        else if (op == 4) excluirEscola();
    } while (op != 0);
}
 
 //crud de ações
 
void imprimirAcao(const Acao *a) {
    Escola *e = buscarEscola(a->idEscola);
    printf("[%d] %s | Escola: %s | Data: %s\n", a->id, a->tema, e ? e->nome : "(removida)", a->data);
    printf("     %s\n", a->descricao);
    printf("     Status: %s | Participantes: %d\n", nomeStatus(a->status), a->participantes);
}
 
void listarAcoes() {
    int i, achou = 0;
    printf("\n--- AÇÕES ---\n");
    for (i = 0; i < totalAcoes; i++) {
        if (!acoes[i].ativo) continue;
        imprimirAcao(&acoes[i]);
        achou = 1;
    }
    if (!achou) printf("Nenhuma ação cadastrada.\n");
}
 
void cadastrarAcao() {
    Acao a;
    if (totalAcoes >= MAX_ACOES) {
        printf("Limite de ações atingido.\n");
        return;
    }
    listarEscolas();
    memset(&a, 0, sizeof(a));
    printf("\n--- NOVA AÇÃO ---\n");
    a.idEscola = lerInteiro("ID da escola: ", 1, MAX_ESCOLAS);
    if (!buscarEscola(a.idEscola)) { printf("Escola não encontrada.\n"); return; }
    if (!lerTexto("Tema (ex.: Vacinação, Saúde bucal): ", a.tema, sizeof(a.tema))) { printf("Tema obrigatório.\n"); return; }
    lerTexto("Descrição: ", a.descricao, sizeof(a.descricao));
    if (!lerTexto("Data (dd/mm/aaaa): ", a.data, sizeof(a.data))) { printf("Data obrigatória.\n"); return; }
    a.status = 0;
    a.participantes = 0;
    a.id = totalAcoes + 1;
    a.ativo = 1;
    acoes[totalAcoes] = a;
    totalAcoes++;
    printf("Ação cadastrada com ID %d.\n", a.id);
}
 
void consultarAcoes() {
    int op, i, achou = 0, id, s;
    char termo[40];
 
    printf("\n--- CONSULTAR AÇÕES ---\n");
    printf("1 - Por ID\n2 - Por escola\n3 - Por status\n4 - Por tema (texto)\n");
    op = lerInteiro("Opção: ", 1, 4);
 
    if (op == 1) {
        Acao *a;
        id = lerInteiro("ID da ação: ", 1, MAX_ACOES);
        a = buscarAcao(id);
        if (a) { imprimirAcao(a); achou = 1; }
    } else if (op == 2) {
        listarEscolas();
        id = lerInteiro("ID da escola: ", 1, MAX_ESCOLAS);
        for (i = 0; i < totalAcoes; i++)
            if (acoes[i].ativo && acoes[i].idEscola == id) { imprimirAcao(&acoes[i]); achou = 1; }
    } else if (op == 3) {
        s = lerInteiro("Status (0-Planejada, 1-Em andamento, 2-Concluída, 3-Cancelada): ", 0, 3);
        for (i = 0; i < totalAcoes; i++)
            if (acoes[i].ativo && acoes[i].status == s) { imprimirAcao(&acoes[i]); achou = 1; }
    } else {
        if (!lerTexto("Parte do tema: ", termo, sizeof(termo))) return;
        for (i = 0; i < totalAcoes; i++)
            if (acoes[i].ativo && strstr(acoes[i].tema, termo) != NULL) { imprimirAcao(&acoes[i]); achou = 1; }
    }
    if (!achou) printf("Nenhuma ação encontrada.\n");
}
 
void atualizarAcao() {
    int id;
    Acao *a;
    char msg[150], novo[100];
 
    listarAcoes();
    id = lerInteiro("\nID da ação a editar (0 = cancelar): ", 0, MAX_ACOES);
    if (id == 0) return;
    a = buscarAcao(id);
    if (!a) { printf("Ação não encontrada.\n"); return; }
 
    printf("(Deixe em branco para manter o valor atual)\n");
    sprintf(msg, "Tema [%s]: ", a->tema);
    if (lerTexto(msg, novo, 40)) strcpy(a->tema, novo);
    sprintf(msg, "Descrição [%s]: ", a->descricao);
    if (lerTexto(msg, novo, 100)) strcpy(a->descricao, novo);
    sprintf(msg, "Data [%s]: ", a->data);
    if (lerTexto(msg, novo, 11)) strcpy(a->data, novo);
    printf("Ação atualizada.\n");
}
 
void atualizarStatus() {
    int id;
    Acao *a;
 
    listarAcoes();
    id = lerInteiro("\nID da ação (0 = cancelar): ", 0, MAX_ACOES);
    if (id == 0) return;
    a = buscarAcao(id);
    if (!a) { printf("Ação não encontrada.\n"); return; }
    printf("Status atual: %s\n", nomeStatus(a->status));
    a->status = lerInteiro("Novo status (0-Planejada, 1-Em andamento, 2-Concluída, 3-Cancelada): ", 0, 3);
    printf("Status atualizado para: %s\n", nomeStatus(a->status));
}
 
void registrarParticipantes() {
    int id;
    Acao *a;
 
    listarAcoes();
    id = lerInteiro("\nID da ação (0 = cancelar): ", 0, MAX_ACOES);
    if (id == 0) return;
    a = buscarAcao(id);
    if (!a) { printf("Ação não encontrada.\n"); return; }
    printf("Participantes atuais: %d\n", a->participantes);
    a->participantes = lerInteiro("Quantidade de participantes: ", 0, 100000);
    printf("Quantidade registrada.\n");
}
 
void excluirAcao() {
    int id;
    Acao *a;
 
    listarAcoes();
    id = lerInteiro("\nID da ação a excluir (0 = cancelar): ", 0, MAX_ACOES);
    if (id == 0) return;
    a = buscarAcao(id);
    if (!a) { printf("Ação não encontrada.\n"); return; }
    a->ativo = 0;
    printf("Ação excluída.\n");
}
 
void menuAcoes(int tipo) {
    int op;
    do {
        printf("\n=== A??ES DO PSE ===\n");
        printf("1 - Cadastrar ação\n");
        printf("2 - Listar todas\n");
        printf("3 - Consultar\n");
        printf("4 - Atualizar dados da ação\n");
        printf("5 - Atualizar status\n");
        printf("6 - Registrar participantes\n");
        if (tipo == 1) printf("7 - Excluir ação\n");
        printf("0 - Voltar\n");
        op = lerInteiro("Opção: ", 0, 7);
        switch (op) {
            case 1: cadastrarAcao(); break;
            case 2: listarAcoes(); break;
            case 3: consultarAcoes(); break;
            case 4: atualizarAcao(); break;
            case 5: atualizarStatus(); break;
            case 6: registrarParticipantes(); break;
            case 7:
                if (tipo == 1) excluirAcao();
                else printf("Acesso negado: apenas administradores.\n");
                break;
        }
    } while (op != 0);
}
 
void gerarResumos() {
    int porStatus[4] = {0, 0, 0, 0};
    int totalA = 0, totalPart = 0;
    int i, e, s, achou = 0;
 
    for (i = 0; i < totalAcoes; i++) {
        if (!acoes[i].ativo) continue;
        totalA++;
        totalPart += acoes[i].participantes;
        if (acoes[i].status >= 0 && acoes[i].status <= 3) porStatus[acoes[i].status]++;
    }
 
    printf("\n========== RESUMO GERAL DO PSE ==========\n");
    printf("Total de ações: %d\n", totalA);
    for (s = 0; s < 4; s++)
        printf("  %-13s: %d\n", nomeStatus(s), porStatus[s]);
    printf("Total de participantes: %d\n", totalPart);
 
    printf("\n--- Por escola ---\n");
    for (e = 0; e < totalEscolas; e++) {
        int qtd = 0, part = 0, concl = 0;
        if (!escolas[e].ativo) continue;
        for (i = 0; i < totalAcoes; i++) {
            if (acoes[i].ativo && acoes[i].idEscola == escolas[e].id) {
                qtd++;
                part += acoes[i].participantes;
                if (acoes[i].status == 2) concl++;
            }
        }
        printf("%-35s ações: %d (concluídas: %d) | participantes: %d\n",
               escolas[e].nome, qtd, concl, part);
        achou = 1;
    }
    if (!achou) printf("Nenhuma escola cadastrada.\n");
    pausar();
}

void menuSistema(int tipo) {
    int op;
 
    limparBuffer();
 
    do {
        printf("\n=====================================\n");
        printf("            MENU DO SISTEMA\n");
        printf("=====================================\n");
        printf("1 - Ações do PSE\n");
        printf("2 - Escolas\n");
        printf("3 - Resumos\n");
        printf("0 - Sair da conta\n");
        op = lerInteiro("Opção: ", 0, 3);
        switch (op) {
            case 1: menuAcoes(tipo); break;
            case 2: menuEscolas(tipo); break;
            case 3: gerarResumos(); break;
        }
    } while (op != 0);
}
 
int main(){
	setlocale(LC_ALL, "Portuguese");
	
	int conta;
	int tipo;
	
	while(3){
	
		printf("\n=====================================\n");
		printf("             SISTEMA DO PSE              ");
		printf("\n=====================================\n");	
		printf("\n01 - criar conta");
		printf("\n02 - entrar ");
		printf("\n03 - sair");
		printf("\nescolha uma opção:");
 
		scanf("%i", &conta);
	
		switch(conta){
			case 1: {
   				Conta novaconta;
   				
    			criarconta(&novaconta);
    			
    			if (totalcontas > 0) {
        		int tipoUsuario = login();
				
       				if (tipoUsuario == 1) {
            			printf("Bem-vindo, Administrador!\n");
       				} else if (tipoUsuario == 2) {
           				printf("Bem-vindo, Profissional da Saúde!\n");
        			}else if (tipoUsuario == 3) {
            			printf("Bem-vindo, Profissional da Educação!\n");
        			}
        			
        			if (tipoUsuario >= 1 && tipoUsuario <= 3) {
        				menuSistema(tipoUsuario);
        			}
    			}
    		break;
			}
			case 2:
				tipo = login();
			
				if(tipo == 1){
					printf("Bem-vindo, administrador!\n");
				}else if(tipo == 2){
					printf("Bem vindo, Proficional da Saúde!");
				}else if(tipo == 3){
					printf("Bem-vindo, Proficional da Educação!");
				}
				
				if (tipo >= 1 && tipo <= 3) {
					menuSistema(tipo);
				}
    		break;
    		case 3:
    			printf("encerrando o sistema...\n");
    			return 0;
			default:
				printf("essa opção não exite!");
		}
	}
	return 0;
}
