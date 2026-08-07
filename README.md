# Material do curso de Desenvolvimento de Software com Agentes e IA Generativa (Dev-IA) de Computação @ UFCG.


## Para Colaboradores

### Configurações

Os dados estão na branch *master* e o código do site está na branch gh-pages.

### Ambiente de "desenvolvimento"

* Este material é feito com o [hugo](https://gohugo.io/) versão v0.75.1/extended darwin/amd64 [download](https://github.com/gohugoio/hugo/releases/tag/v0.75.1).

* Você, colaborador externo, não deve se preocupar com a branch gh-pages. Tudo que você precisa para produzir conteúdo e colocar o site no ar localmente pode ser feito com
o conteúdo que está na branch master.

* Para gerar o site localmente em ambiente de dev, basta executar `hugo server` com os dados clonados da branch master e acessar o link que ele coloca no final da mensagem. Normalmente é http://localhost:1313/dev-ia/.
Sempre que `hugo server` é executado, ele gera uma nova versão do site na pasta `public`.

### Produzindo conteúdo

Se quiser produzir um conteúdo de algum assunto, basta criar arquivo .md dentro da pasta `content/posts`. Veja os exemplos já feitos lá. Se for adicionar figuras, cria-se um diretório com o mesmo
nome do arquivo dentro de `content/posts`. Veja também a página [Contribua](/dev-ia/contribua/).

### Contribuindo

Depois de produzir o conteúdo, basta enviar o Pull Request com o arquivo md e o diretório de figuras, caso tenha adicionado algum.

## Para admin

* Para subir uma nova versão do site: `./deploy.sh "mensagem"`

### Set up inicial

<pre>
git clone https://github.com/joaoarthurbm/dev-ia.git
echo "public" >> .gitignore
git checkout --orphan gh-pages
git reset --hard
git commit --allow-empty -m "Initializing gh-pages branch"
git config pull.rebase true
git pull
git push origin gh-pages
git checkout master
</pre>

<pre>
rm -rf public
git worktree add -B gh-pages public origin/gh-pages
</pre>

**Nota:** o repositório `dev-ia` ainda não existe no GitHub; crie-o (público) antes de executar o set up inicial acima. Depois é só rodar `./deploy.sh` para publicar em https://joaoarthurbm.github.io/dev-ia/.