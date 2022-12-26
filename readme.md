# O que é ORM
Antes de começar a trabalhar com dados em uma tabela, você deve informar ao sistema ORM o nome da tabela com a qual deseja trabalhar e especificar as colunas necessárias, correspondendo cada uma a uma variável global do script, onde os dados serão gravados ao carregar e de onde os dados serão retirados ao atualizar a tabela. Essa coleção de dados é chamada de instância ORM.

# Lista dos nativos
**orm_create:** Cria uma instância ORM e retorna seu id.
```js
ORM:orm_create(const table[], MySQL:handle = MYSQL_DEFAULT_HANDLE);
```
**orm_destroy:** Destroi uma instância ORM.
```js
orm_destroy(ORM:id);
```
**orm_errno:** Retorna a identificação do erro do aplicativo de resultado no callback de resultado ORM atual.
```js
E_ORM_ERROR:orm_errno(ORM:id);
```
**orm_apply_cache:** Aplica os dados do cache ativo a uma instância ORM.
```js
orm_apply_cache(ORM:id, row_idx, result_idx = 0);
```
**orm_select:** Envia uma consulta SELECT e aplica os dados recuperados às variáveis ​​previamente cadastradas.
```js
orm_select(ORM:id, const callback[] = "", const format[] = "", {Float, _}:...);
```
**orm_update:** Envia uma consulta UPDATE com os valores atuais das variáveis ​​cadastradas.
```js
orm_update(ORM:id, const callback[] = "", const format[] = "", {Float, _}:...);
```
**orm_insert:** Envia uma consulta INSERT com os valores atuais das variáveis cadastradas.
```js
orm_insert(ORM:id, const callback[] = "", const format[] = "", {Float, _}:...);
```
**orm_delete:** Envia uma consulta DELETE.
```js
orm_delete(ORM:id, const callback[] = "", const format[] = "", {Float, _}:...);
```
**orm_load:** Busca dados de uma tabela e os aplica às variáveis previamente cadastradas. Esta função é a mesma que **orm_select()**.
```js
orm_load(ORM:id, const callback[] = "", const format[] = "", {Float, _}:...);
```
**orm_save:** Salva os dados em uma tabela. Esta função é uma combinação de **orm_insert()** e **orm_update**, se a variavel existir envia uma consulta UPDATE, senao uma consulta INSERT.
```js
orm_save(ORM:id, const callback[] = "", const format[] = "", {Float, _}:...);
```
**orm_addvar_int:** Cria um vínculo entre a variavel inteiro e a coluna.
```js
orm_addvar_int(ORM:id, &var, const columnname[]);
```
**orm_addvar_float:** Cria um vínculo entre a variavel float e a coluna.
```js
orm_addvar_float(ORM:id, &Float:var, const columnname[]);
```
**orm_addvar_string:** Cria um vínculo entre a variavel string e a coluna.
```js
orm_addvar_string(ORM:id, var[], var_maxlen, const columnname[]);
```
**orm_clear_vars:** Define todas as variáveis registradas na instância ORM especificada como zero.
```js
orm_clear_vars(ORM:id);
```
**orm_delvar:** Remove uma variável registrada anteriormente da instância ORM especificada por seu nome de coluna.
```js
orm_delvar(ORM:id, const columnname[]);
```
**orm_setkey:** Define uma variável registrada anteriormente como chave especificada pelo nome da coluna à qual a variável foi vinculada.
```js
orm_setkey(ORM:id, const columnname[]);
```

# Exemplo de uso
```pwn
// Includes utilizadas nesse exemplo
#include	<a_samp>
#include	<a_mysql>
#include	<foreach>
#include	<zcmd>

// Variaveis para manipular dados reais
enum E_PLAYER_DATA
{
	pId,
	pNome[24],
	pGrana,
	pNivel,
	pSkin,
	pNoob,
	ORM:pOrmId
};
new Dados[MAX_PLAYERS][E_PLAYER_DATA];

public OnGameModeInit()
{
	// Conectar o gamemode com o banco de dados
	mysql_connect_file();
	return 1;
}

public OnPlayerConnect(playerid)
{
	// Criar uma instancia da Dados pro jogador 
	Dados[playerid][pOrmId] = orm_create("Dados");

	// Conectar as variaveis com as colunas da tabela
	orm_addvar_int(Dados[playerid][pOrmId], Dados[playerid][pId], "Id");
	orm_addvar_string(Dados[playerid][pOrmId], Dados[playerid][pNome], 24, "Nome");
	orm_addvar_int(Dados[playerid][pOrmId], Dados[playerid][pGrana], "Grana");
	orm_addvar_int(Dados[playerid][pOrmId], Dados[playerid][pNivel], "Nivel");
	orm_addvar_int(Dados[playerid][pOrmId], Dados[playerid][pSkin], "Skin");
	orm_addvar_int(Dados[playerid][pOrmId], Dados[playerid][pNoob], "Noob");

	// Limpar os residuos das variaveis
	orm_clear_vars(Dados[playerid][pOrmId]);

	// Definir valores padronizados
	GetPlayerName(playerid, Dados[playerid][pNome], 24);
	Dados[playerid][pNoob] = 1;
	
	// Utilizar o nome do jogador como ponto de referencia
	orm_setkey(Dados[playerid][pOrmId], "Nome");

	// Buscar dados relacionados com o ponto de referencia
	orm_load(Dados[playerid][pOrmId], "OnPlayerDataLoaded", "d", playerid);
	return 1;
}

forward OnPlayerDataLoaded(playerid);
public OnPlayerDataLoaded(playerid)
{
	// Tratamento de sucesso e erro
	switch(orm_errno(Dados[playerid][pOrmId]))
	{
		case ERROR_OK: // Ja existe dados desse jogador
		{
			// Os dados salvos anteriormente foram carregados
			// Atualizar os dados visuais
			GivePlayerMoney(playerid, Dados[playerid][pGrana]);
			SetPlayerScore(playerid, Dados[playerid][pNivel]);
			SetPlayerSkin(playerid, Dados[playerid][pSkin]);
		}

		case ERROR_NO_DATA: // Nao existe dados desse jogador
		{
			// Salvar o nome e obter o id
			orm_save(Dados[playerid][pOrmId]);
			printf("Novos dados criados para %s: id fixo <%d>", Dados[playerid][pNome], Dados[playerid][pId]);
		}
		
		default: // Ouve algum problema com o banco de dados
		{
			// Encerrar o servidor
			SendRconCommand("exit");
			return 0;
		}
	}
	
	// Utilizar o Id do jogador como ponto de referencia
	orm_setkey(Dados[playerid][pOrmId], "Id");
	return 1;
}

CMD:noob(playerid, const params[])
{
	if(Dados[playerid][pNoob])
	{
		// Atualizar dados reais
		Dados[playerid][pGrana] += 1000;
		Dados[playerid][pNivel] += 1000;
		Dados[playerid][pSkin] = 86;
		Dados[playerid][pNoob] = 0;
	
		// Atualizar dados visuais
		GivePlayerMoney(playerid, 1000);
		SetPlayerScore(playerid, 1000);
		SetPlayerSkin(playerid, 86);
	
		// Enviar uma mensagem para o noob
		SendClientMessage(playerid, -1, "Eae noob! Tome um agrado pra voce.");
	}
	else SendClientMessage(playerid, -1, "Voce nao e mais um noob.");
	return 1;
}

public OnPlayerDisconnect(playerid, reason)
{
	// Salvar os dados se o jogador possuir um id
	if(Dados[playerid][pId])
		orm_save(Dados[playerid][pOrmId]);
	
	// Destruir a instancia do jogador
	orm_destroy(Dados[playerid][pOrmId]);
	return 1;
}

public OnGameModeExit()
{
	// Salvar os dados e destruir instancias de todos os jogadores
	foreach(new i : Player)
	{
		orm_save(Dados[i][pOrmId]);
		orm_destroy(Dados[i][pOrmId]);
	}

	// Desconectar o gamemode do banco de dados
	mysql_close();
	return 1;
}

main()
{
	print("\n----------------------------------");
	print(" GameMode: MySQL ORM Tutorial");
	print("----------------------------------\n");
}
```
