# JCL

**Versão atual:** commit 4ec204493293be0630eafae5d57f49acc1073486 do branch **master** de 15/11/2015. 

### Passos para atualizar a JCL

1. Execute o instaldor do JCL (install.bat). Em cada aba associada a versão do Delphi:
	* Desmarque a opção *Wrapper options*
	* Desmarque a opção *Environment*
	* Desmarque a opção *Make library units*
	* Desmarque a opção *Packages*
	* Desmarque a opção *Enable thread safe code* (o Engine utiliza locks próprios para sincronizar o uso das classes da JCL)
	* Configure a precisão de ponto flutuante para DOUBLE

2. Copei os diretórios abaixo para o diretório jcl do repositório: 
	* jcl\source\common
	* jcl\source\include
	* jcl\source\vcl
	* jcl\source\windows

3. Remova os diretórios:
	* jcl\windows\obj (o objetivo é garantir que a ZLIB não será linkada estaticamente)
	* jcl\include\jedi\.git

4. Nos arquivos jcl\include\jclNN*.inc, adicionar o código abaixo no final do arquivo. Este workaround deverá ser eliminado quando for migrado para o Delphi XE2 ou superior.

		{$IF CompilerVersion < 23}
		{$DEFINE JCL_PCRE}
		{$DEFINE JCL_PCRE_8}
		{$DEFINE PCRE_LINKONREQUEST}
		{$DEFINE PCRE_8}
		{$UNDEF PCRE_RTL}	
		{$UNDEF PCRE_16}
		{$IFEND}

5. No arquivo jcl\source\common\JclAnsiStrings.pas, tornar os métodos beginUpdate e endUpdate da classe TJclAnsiStrings virtuais. Este é um workaround para uma implementação incompleta dessa classe. Esse workaround poderá ser eliminado quando a IDE for desligada ou quando TJclAnsiStrings implementar a API correta de TStrings, com os eventos onChange e onChanging.

6. Verificar se a implementação do método *TJclAnsiStringList.AddObject* foi corrigida para tratar corretamente a opção *dupAccept*. Na versão atual, *Result* está ficando indefinido, coforme pode ser observado no código abaixo. Caso o defeito tenha sido corrigido, deverá ser removido o workaround em
*TIclAnsiStringList.AddObject*.

		function TJclAnsiStringList.AddObject(const S: AnsiString; AObject: TObject): Integer;
		begin
		  if not Sorted then
		  begin
		    Result := Count;
		  end
		  else
		  begin
		    case Duplicates of
		      dupAccept: ;
		      dupIgnore:
		        if Find(S, Result) then
		          Exit;
		      dupError:
		        if Find(S, Result) then
		          Error(@SDuplicateString, 0);
		    end;
		  end;

		  InsertObject(Result, S, AObject);
		end;
