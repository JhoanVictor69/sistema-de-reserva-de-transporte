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

## Documentação

`docs/01_requisitos.md` · `docs/02_justificativa_tecnica.md` · `docs/03_evidencias_testes.md` · `docs/04_roteiro_apresentacao.md` · `docs/05_guia_estudo_codigo.md`

> `DEBUG=True` e a `SECRET_KEY` do `settings.py` servem somente para uso acadêmico local.
