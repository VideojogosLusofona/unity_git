<!--
Guia para uso de Git em projetos Unity (c) by Nuno Fachada and contributors

Guia para uso de Git em projetos Unity is licensed under a
Creative Commons Attribution-ShareAlike 4.0 International Public License.

You should have received a copy of the license along with this
work. If not, see <https://creativecommons.org/licenses/by-sa/4.0/>.
-->

[![CC BY-SA 4.0](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

# Guia para uso de Git em projetos Unity

Este guia assume que o seguinte software está instalado num sistema Windows 10:

* [Unity](https://unity3d.com/)
* [Git](https://git-scm.com/) e [Git LFS](https://git-lfs.github.com/)
* [KDiff3](http://kdiff3.sourceforge.net/)
* [Notepad++](https://notepad-plus-plus.org/)

Alguns detalhes poderão ser ligeiramente diferentes em Mac e Linux.

## Configuração dos projetos Unity

Confirmar as seguintes definições em cada novo projeto no Unity:

* Edit &#8594; Project Settings &#8594; Version Control
  1. Mode &#8594; Visible Meta Files
* Edit &#8594; Project Settings &#8594; Editor
  1. Asset Serialization, Mode &#8594; Force Text
* Edit &#8594; Project Settings &#8594; Player
  1. API Compatibility Level &#8594; .NET Standard 2.0

## Configuração do Git

### Ficheiros de configuração

Colocar na pasta raiz do projeto Unity os seguintes ficheiros:

* [`.gitignore`](.gitignore) -
  Indica ficheiros temporários a ignorar, ou seja, que não serão guardados
  no repositório.
* [`.gitattributes`](.gitattributes) -
  Entre outras coisas, indica ficheiros a guardar em modo Git LFS,
  nomeadamente ficheiros binários (imagens, sons, texturas, vídeos, etc).

A forma mais simples de o fazer é abrir cada um dos *links* no *browser* e
fazer CTRL+S para salvar cada um dos ficheiros. Os ficheiros devem ser
guardados na pasta raiz do projeto Unity e devem ter exatamente os nomes
`.gitignore` e `.gitattributes`.

### Inicializar repositório

Abrir Git Bash na pasta do projeto (ou fazer `cd` até à pasta do projeto), e
inicializar o repositório Git com o seguinte comando:

* `git init`

Inicializar Git LFS para o repositório criado:

* `git lfs install`

## Configuração para gestão de *merge conflicts*

### Configuração ao nível do Git

Na pasta do projeto editar ficheiro `config` na pasta `.git` e colocar o
seguinte no fim do mesmo (confirma que o caminho completo até à ferramenta
`UnityYAMLMerge` está correto - nota que são necessário duas barras entre
cada pasta):

```ini
[merge]
	tool = unityyamlmerge
[mergetool "unityyamlmerge"]
	trustExitCode = false
	cmd = 'C:\\Program Files\\Unity\\Hub\\Editor\\2018.3.0f2\\Editor\\Data\\Tools\\UnityYAMLMerge.exe' merge -p "$BASE" "$REMOTE" "$LOCAL" "$MERGED"
[mergetool]
	keepBackup = false
```

Se a instalação do Unity onde se encontra a ferramenta `UnityYAMLMerge` for
removida, é necessário atualizar o ficheiro `.git\config` para procurar a
ferramenta na nova instalação do Unity.

### Configuração ao nível do Unity

A mesma pasta que contém a ferramenta `UnityYAMLMerge` contém também o seu
ficheiro de configuração, de nome `mergespecfile.txt`. Abrir este ficheiro com
o Notepad++ e substituir as linhas:

```text
unity use "%programs%\YouFallbackMergeToolForScenesHere.exe" "%l" "%r" "%b" "%d"
prefab use "%programs%\YouFallbackMergeToolForPrefabsHere.exe" "%l" "%r" "%b" "%d"
```

por

```text
unity use "%programs%/KDiff3/kdiff3.exe" "%b" "%l" "%r" -o "%d"
prefab use "%programs%/KDiff3/kdiff3.exe" "%b" "%l" "%r" -o "%d"
* use "%programs%/KDiff3/kdiff3.exe" "%b" "%l" "%r" -o "%d"
```

São necessárias permissões de administrador para alterar o ficheiro. Ao guardar
o ficheiro o Notepad++ pergunta se queres abrir o mesmo com permissões de
administrador. Confirma, volta a inserir as modificações e guarda o ficheiro.

Se a instalação do Unity onde se encontra o ficheiro `mergespecfile.txt` for
removida, é necessário modificar o ficheiro `mergespecfile.txt` da nova
instalação do Unity de igual forma.

### Fazer *merge* de ramos diferentes em projetos Unity

Por exemplo, se estivermos no ramo `master` e quisermos fazer *merge* do ramo
`new-boss`, vamos executar o seguinte comando `git merge new-boss`. Se
existirem conflitos que o Git não conseguir resolver, o resultado vai ser algo
do género:

```text
Auto-merging Assets/Scenes/MyScene.unity
CONFLICT (content): Merge conflict in Assets/Scenes/MyScene.unity
Automatic merge failed; fix conflicts and then commit the result.
```

Para resolver os conflitos, vamos invocar a ferramenta configurada para o
efeito com o comando `git mergetool`. A maioria dos conflitos é resolvida
automaticamente. No caso dos conflitos não resolvidos será aberta uma janela do
KDiff3, tendo o utilizador apenas de especificar, para cada conflito, se quer
utilizar as modificações do ramo `master` ou do ramo `new-boss`, e guardar
essas escolhas. Os conflitos são marcados como resolvidos e basta fazer
`git commit` para finalizar o *merge*.

## Configurar Git para usar Notepad++ nas mensagens de *commit*

Por omissão o Git usa o editor Vim para as mensagens de *commit*. No entanto, é
muito mais prático usar um editor como o Notepad++. O seguinte comando
configura o Git para usar o Notepad++ nas mensagens de *commit* (assume-se que
o Notepad++ está instalado na pasta indicada, caso contrário alterar em
conformidade):

```powershell
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

Uma vez que é especificada a opção `--global`, basta executar este comando uma
vez para que o Notepad++ fique configurado para todos os repositórios Git de um
dado utilizador no mesmo PC.

## Referências

* [Unity User Manual - Smart Merge](https://docs.unity3d.com/Manual/SmartMerge.html)
* [Setting up SourceTree to merge unity3d scenes with UnityYAMLMerge](https://stackoverflow.com/questions/36566436/setting-up-sourcetree-to-merge-unity3d-scenes-with-unityyamlmerge/45738772)
* [Smart Merging Unity](https://gist.github.com/diego-augusto/876bd825730755a4d969d5e436464858)
* [Unity, SourceTree and Merge Conflicts](https://www.youtube.com/watch?v=EQB-N-ClO9g)
* [Git mergetool generates unwanted .orig files](https://stackoverflow.com/questions/1251681/git-mergetool-generates-unwanted-orig-files)
* [How to set Notepad++ as the default Git editor for commits instead of Vim](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/How-to-set-Notepad-as-the-default-Git-editor-for-commits-instead-of-Vim)
