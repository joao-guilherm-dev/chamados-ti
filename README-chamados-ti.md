# Central de Chamados — configurar Firebase e publicar no GitHub Pages

## 1. Criar o projeto Firebase (gratuito)

1. Acesse https://console.firebase.google.com/ e clique em **Adicionar projeto**.
2. Dê um nome (ex: `chamados-ti`) e siga o assistente (pode desativar o Google Analytics, não é necessário).
3. Dentro do projeto, vá em **Compilação → Firestore Database → Criar banco de dados**.
   - Escolha **modo de produção**.
   - Selecione uma região (ex: `southamerica-east1` — São Paulo).
4. Vá em **Regras** (aba dentro do Firestore) e substitua pelo conteúdo abaixo, depois clique em **Publicar**:

   ```
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
   ```

   Isso deixa qualquer pessoa **abrir** e **ler** chamados (é a ideia — não tem login),
   mas só permite **alterar o status** de um chamado já existente, sem apagar ou reescrever o resto.
   Não é uma segurança de nível corporativo, mas evita que alguém apague tudo por acidente
   ou reescreva um chamado inteiro pela API.

5. Ainda no console, clique no ícone **⚙ → Configurações do projeto**, role até **Seus apps**,
   clique no ícone `</>` (Web) e registre um app (não precisa marcar Hosting).
6. Copie o objeto `firebaseConfig` que aparece — algo assim:

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "chamados-ti-xxxx.firebaseapp.com",
     projectId: "chamados-ti-xxxx",
     storageBucket: "chamados-ti-xxxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```

   Essas chaves são **públicas por natureza** (identificam o projeto, não dão acesso de admin) —
   pode colar e commitar sem problema, quem protege os dados são as **regras do Firestore** do passo 4.

## 2. Colar a configuração no arquivo

Abra `chamados-ti-firebase.html`, procure o bloco `const firebaseConfig = {...}` perto do
início do `<script type="module">` e substitua pelos valores copiados no passo anterior.

Se quiser, troque também o código de acesso do painel (`const PIN = '1234';`) para outro número.

## 3. Publicar no GitHub Pages

Se for dentro do repositório do **sentinel-ops-toolkit**:

```bash
# na raiz do repositório
mkdir -p chamados-ti
cp chamados-ti-firebase.html chamados-ti/index.html
git add chamados-ti
git commit -m "Adiciona site de chamados de TI"
git push
```

Depois, no GitHub: **Settings → Pages → Branch** → selecione `main` e a pasta `/chamados-ti`
(ou `/root` se preferir subir sozinho num repositório novo) → **Save**.

Em alguns minutos o site fica disponível em algo como:
`https://<seu-usuario>.github.io/<repositorio>/chamados-ti/`

## 4. Testar

- Abra o link em um celular e registre um chamado de teste.
- Abra o mesmo link em outro aparelho (ou no navegador do computador), entre no painel com o
  código, e o chamado deve aparecer em tempo real — não precisa recarregar a página.

## Limites do plano gratuito (Spark)

Mais do que suficiente para um helpdesk interno: 50 mil leituras e 20 mil gravações por dia,
1 GiB de armazenamento. Se um dia crescer muito, o Firebase avisa antes de cobrar.
