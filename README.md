# COSEG Mobilidade — Reserva de Veículos do Porto do Itaqui

Projeto do **PBL 1 — Agenda em Conflito** (Programação para Web, UNDB, prof. Danilo Costa).
Camada de servidor em Python/Django com SQLite: cadastro da frota, reservas com CRUD, validação de
capacidade, data, horários e conflitos, API JSON e uma interface simples para demonstração.

## O que existe no projeto

```text
coseg_mobilidade/
├── manage.py
├── requirements.txt
├── README.md
├── config/                  # configurações do projeto (settings, urls, wsgi, asgi)
├── reservas/                # app principal
│   ├── models.py            # Veiculo e Reserva
│   ├── regras.py            # regras de negócio (capacidade, data, horários, conflito, disponibilidade)
│   ├── forms.py             # validação de campos obrigatórios e tipos
│   ├── views.py             # telas HTML
│   ├── api.py               # endpoints JSON
│   ├── urls.py
│   ├── admin.py
│   ├── tests.py             # 52 testes automatizados
│   ├── migrations/
│   └── management/commands/carregar_dados.py
├── templates/               # páginas HTML
├── static/css/estilo.css
└── docs/                    # documentação acadêmica
```

## Instalação no Windows (VS Code + PowerShell)

1. **Extrair o ZIP.** Clique com o botão direito em `coseg_mobilidade.zip` → *Extrair tudo…*. Escolha uma pasta simples, por exemplo `C:\Projetos\`.
2. **Abrir a pasta correta no VS Code.** *Arquivo → Abrir Pasta…* e selecione a pasta `coseg_mobilidade` que contém o arquivo `manage.py` (não a pasta de fora, caso o ZIP crie uma pasta dentro de outra).
3. **Abrir o terminal.** No VS Code: *Terminal → Novo Terminal* (deve ser PowerShell e já abrir na pasta do projeto). Confirme com `dir`: o `manage.py` precisa aparecer.
4. **Verificar o Python.**
   ```powershell
   py --version
   ```
   Deve mostrar Python 3.10 ou superior. Se o comando não existir, instale em python.org marcando *"Add python.exe to PATH"*.
5. **Criar o ambiente virtual.**
   ```powershell
   py -m venv .venv
   ```
6. **Ativar o ambiente virtual** (opcional; os comandos abaixo usam o caminho completo e funcionam sem ativar):
   ```powershell
   .\.venv\Scripts\Activate.ps1
   ```
   Se aparecer erro de "execução de scripts desabilitada", execute antes `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` (vale só para este terminal) ou simplesmente pule este passo.
7. **Instalar as dependências.**
   ```powershell
   .\.venv\Scripts\python.exe -m pip install -r requirements.txt
   ```
8. **Criar as tabelas (migrations).**
   ```powershell
   .\.venv\Scripts\python.exe manage.py migrate
   ```
9. **Cadastrar a frota e as 4 reservas do enunciado.**
   ```powershell
   .\.venv\Scripts\python.exe manage.py carregar_dados
   ```
   Pode ser repetido sem duplicar. Para cadastrar só os 10 veículos: `manage.py carregar_dados --apenas-frota`.
10. **Iniciar o servidor.**
    ```powershell
    .\.venv\Scripts\python.exe manage.py runserver
    ```
11. **Abrir no navegador:** http://127.0.0.1:8000/ (para encerrar o servidor, `Ctrl + C` no terminal).
12. **Executar os testes** (em outro terminal, ou com o servidor parado):
    ```powershell
    .\.venv\Scripts\python.exe manage.py test
    ```
    Deve terminar com `Ran 52 tests ... OK`.

Opcional — acesso ao Django Admin (http://127.0.0.1:8000/admin/):
`.\.venv\Scripts\python.exe manage.py createsuperuser`

## Telas

| Endereço | Conteúdo |
|---|---|
| `/` | Início: quantidade de veículos e reservas |
| `/veiculos/` | Frota (código, categoria, capacidade) |
| `/reservas/` | Lista com filtros por data e veículo, editar e excluir |
| `/reservas/nova/` | Formulário de reserva (mensagens de erro específicas) |
| `/disponibilidade/` | Consulta de veículos livres antes de reservar |

## API JSON

| Método e rota | Função | Sucesso |
|---|---|---|
| `GET /api/veiculos/` | Lista veículos | 200 |
| `GET /api/veiculos/<id>/` | Consulta um veículo | 200 / 404 |
| `GET /api/reservas/` | Lista reservas (`?data=AAAA-MM-DD&veiculo=VL-01`) | 200 |
| `GET /api/reservas/<id>/` | Consulta uma reserva | 200 / 404 |
| `POST /api/reservas/` | Cria reserva | 201 / 400 |
| `PUT /api/reservas/<id>/` | Atualiza (envia todos os campos) | 200 / 400 / 404 |
| `DELETE /api/reservas/<id>/` | Exclui | 204 / 404 |
| `GET /api/disponibilidade/` | `?data=…&saida=HH:MM&retorno=HH:MM[&passageiros=N]` | 200 / 400 |
| `GET /api/csrf/` | Entrega o token CSRF (necessário para POST/PUT/DELETE) | 200 |

Corpo de uma reserva (JSON). O campo `veiculo` recebe o **código** do veículo:

```json
{
  "solicitante": "Maria Souza", "setor": "Operações", "atividade": "Reunião externa",
  "origem": "Porto do Itaqui", "destino": "Centro Administrativo",
  "data": "2026-12-10", "horario_saida": "08:00", "horario_retorno": "10:00",
  "passageiros": 3, "veiculo": "VL-01", "observacoes": ""
}
```

Erros têm sempre o formato `{"erro": "resumo", "erros": {"campo": ["mensagem"]}}`; conflitos aparecem em `erros.geral`.

### Testando a API (POST, PUT e DELETE com CSRF ativo)

A proteção CSRF do Django **não foi desativada**. Isso significa que, fora do navegador, uma requisição POST/PUT/DELETE precisa de (1) o cookie `csrftoken` e (2) o mesmo valor no cabeçalho `X-CSRFToken`. O endpoint `GET /api/csrf/` entrega os dois. Sem eles, a resposta é `403` em JSON.

Requisições `GET` não precisam de nada: abra `http://127.0.0.1:8000/api/reservas/` no navegador.

Exemplo em PowerShell (o servidor deve estar rodando em outro terminal):

```powershell
$base = "http://127.0.0.1:8000"
$sessao = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$token = (Invoke-RestMethod "$base/api/csrf/" -WebSession $sessao).csrfToken
$cab = @{ "X-CSRFToken" = $token }

$corpo = @{
  solicitante = "Maria Souza"; setor = "Operações"; atividade = "Reunião externa"
  origem = "Porto do Itaqui"; destino = "Centro Administrativo"
  data = "2026-12-10"; horario_saida = "08:00"; horario_retorno = "10:00"
  passageiros = 3; veiculo = "VL-01"
} | ConvertTo-Json
$bytes = [System.Text.Encoding]::UTF8.GetBytes($corpo)

# Criar (POST)
try { Invoke-RestMethod "$base/api/reservas/" -Method Post -WebSession $sessao -Headers $cab -ContentType "application/json; charset=utf-8" -Body $bytes }
catch { $_.ErrorDetails.Message }   # mostra o JSON de erro (400, 403...)

# Atualizar (PUT) e excluir (DELETE) a reserva de id 5:
# Invoke-RestMethod "$base/api/reservas/5/" -Method Put -WebSession $sessao -Headers $cab -ContentType "application/json; charset=utf-8" -Body $bytes
# Invoke-RestMethod "$base/api/reservas/5/" -Method Delete -WebSession $sessao -Headers $cab
```

Repita o mesmo POST para ver o conflito de horário (400). Esse fluxo foi verificado com `curl` em Linux; os comandos em PowerShell seguem a mesma lógica, mas não foram executados em Windows durante a elaboração.

## Regras implementadas

- Capacidade: leves até 4, coletivos até 18; acima de 18, zero ou negativo são rejeitados.
- Data: não pode ser anterior à data atual no fuso `America/Fortaleza`.
- Horários: retorno deve ser posterior à saída.
- Conflito: mesmo veículo, mesma data e `nova_saida < retorno_existente` **e** `novo_retorno > saida_existente`. **Decisão sobre o limite:** uma reserva que termina às 10:00 e outra que começa às 10:00 **não** conflitam (comparações estritas).
- Campos obrigatórios, tipos inválidos e veículo inexistente geram mensagens específicas.

## Problemas comuns

| Sintoma | Solução |
|---|---|
| `'py' não é reconhecido` | Instale o Python marcando *Add to PATH*, ou tente `python --version`. |
| `ModuleNotFoundError: No module named 'django'` | Rode os comandos com `.\.venv\Scripts\python.exe` ou ative o ambiente virtual e reinstale (passo 7). |
| `can't open file ... manage.py` | O terminal está na pasta errada. Use `cd` até a pasta que contém `manage.py`. |
| `Activate.ps1 não pode ser carregado` | Veja o passo 6 (ou use o caminho completo do Python, sem ativar). |
| Erro ao instalar o Django (versão do Python) | O Django 5.2 exige Python 3.10+. Atualize o Python. |
| `Error: That port is already in use` | Use outra porta: `manage.py runserver 8001`. |
| Página sem veículos | Execute `manage.py carregar_dados` (passo 9). |
| `no such table` | Execute `manage.py migrate` (passo 8). |
| Quer recomeçar do zero | Pare o servidor, apague o arquivo `db.sqlite3` e repita os passos 8 e 9. |

## Documentação

`docs/01_requisitos.md` · `docs/02_justificativa_tecnica.md` · `docs/03_evidencias_testes.md` · `docs/04_roteiro_apresentacao.md` · `docs/05_guia_estudo_codigo.md`

> `DEBUG=True` e a `SECRET_KEY` do `settings.py` servem somente para uso acadêmico local.
