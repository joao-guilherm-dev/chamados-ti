Central de Chamados · TI

Site simples para abertura e acompanhamento de chamados de TI. Feito para funcionar bem no celular, sem precisar de login complicado nem servidor próprio — só um arquivo HTML hospedado no GitHub Pages e um banco Firestore gratuito por trás.

O que ele faz
Abrir chamado: qualquer pessoa preenche nome, setor, categoria, prioridade e descrição do problema, e recebe um número de protocolo.
Painel (TI): atrás de um código de acesso simples, mostra todos os chamados em tempo real, com filtro por status (Aberto / Em andamento / Resolvido) e botões para mudar o status de cada um.
Atualização em tempo real: um chamado aberto em um celular aparece na hora no painel de quem está acompanhando, em qualquer outro aparelho.
Tecnologia
HTML, CSS e JavaScript puro (nenhuma build, nenhuma dependência de npm).
Firebase Firestore como banco de dados, usado direto pelo navegador via SDK modular (v10).
GitHub Pages para hospedagem estática.
Estrutura
.
├── index.html   # o site inteiro (formulário + painel)
└── README.md
Configuração
Crie um projeto no Firebase Console e ative o Firestore Database (modo produção).
Nas Regras do Firestore, cole:
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /chamados/{ticketId} {
         allow read: if true;
         allow create: if request.resource.data.keys().hasAll(['nome','desc','categoria','prioridade','status']);
         allow update: if request.resource.data.diff(resource.data).affectedKeys()
                          .hasOnly(['status','atualizadoEm']);
       }
     }
   }
Em Configurações do projeto → Seus apps, registre um app Web e copie o objeto firebaseConfig.
Cole esse objeto no início do <script type="module"> dentro de index.html, no lugar dos valores "COLE_AQUI".
(Opcional) Troque o código de acesso do painel, definido em const PIN = '1234';.

As chaves do firebaseConfig são públicas por natureza — quem protege os dados de verdade são as regras do passo 2, não essas chaves.

Publicar
bash
git add .
git commit -m "Adiciona site de chamados de TI"
git push

Depois, no repositório: Settings → Pages → Branch → escolha main e a pasta onde está o index.html → Save. O site fica disponível em https://SEU-USUARIO.github.io/SEU-REPO/.

Limites do plano gratuito do Firebase

50 mil leituras e 20 mil gravações por dia, 1 GiB de armazenamento — de sobra para um helpdesk interno.

Aviso

O código de acesso do painel é só para separar a visão da equipe de TI da visão de quem abre chamado — não é uma camada de segurança de verdade. A proteção real dos dados está nas regras do Firestore.
