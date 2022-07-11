# Workshop B2C

## Criar um tenant B2C para testes
Para a finalidade de testes, um tenant do Azure AD B2C pode ser criado. Até o momento da escrita deste documento, o B2C suporta até 50.000 usuários mensais ativos de forma gratuita.

- [Criar um tenant ADB2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant )
- [Precificação ADB2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/billing)

## App Registration
Uma App Registration permite abstrair o acesso das aplicações ao B2C. Desta forma é possível definir e controlar quais são os clientes que estão acessando o provedor de identidade. 

- [Criar um app](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-applications?tabs=app-reg-ga)

Configuração|Valor
-|-
Name|b2c-workshop-app
Supported account types|Accounts in any identity provider or organizational directory (for authenticating users with user flows)
Redirect URI (recommended)|Web - https://jwt.ms
Authentication|Access tokens (used for implicit flows) ON
> Após criado, adicionar um client secret em Client & Secrets




### Requisitando um token de aplicação
```bash
curl --location --request POST 'https://login.microsoftonline.com/patrickreinanworkshopb2c.onmicrosoft.com/oauth2/v2.0/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=85187823-cdb4-4ac0-809b-c6c8b239bc06' \
--data-urlencode 'client_secret=client_secret_value' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'scope=.default'
```

## User Flows e Custom Policies
- **User Flows**: modelo que permite a criação de interfaces com o usuário de forma fácil, porém menos flexível.
- **Custom Policies**: modelo baseado em configuração declarativa (XML) que permite customizar os fluxos de interação com o usuário

## Criando um User Flow
Configuração|Valor
-|-
Create a user flow|Sign up and sign in
Version|Recommended
Name|WORKSHOP_SIGNUP_SIGNIN
Local Accounts|Email signup
User attributes and token claims| Collect attribute e Return claim: Display Name

> Crie um usuário e tente fazer o login novamente.


## Criar um usuário
Configuração|Valor
-|-
Select Template|Create user
Identity - User Name|johndoe
Identity - Name|John
Password|Let me create the password
Initial Password|1qa2ws3ed!QA@WS#ED
Settings - Usage Location|Brazil



## User Flows e Custom Policies
- **User Flows**: modelo que permite a criação de interfaces com o usuário de forma fácil, porém menos flexível.
- **Custom Policies**: modelo baseado em configuração declarativa (XML) que permite customizar os fluxos de interação com o usuário


### Requisitando um token de usuário