# Workshop B2C

## Criar um tenant B2C para testes
Para a finalidade de testes, um tenant do Azure AD B2C pode ser criado. Até o momento da escrita deste documento, o B2C suporta até 50.000 usuários mensais ativos de forma gratuita.

- [Criar um tenant ADB2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-tenant )
- [Precificação ADB2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/billing)

## App Registration
Um App Registration permite abstrair o acesso das aplicações ao B2C. Desta forma é possível definir e controlar quais são os clientes que estão acessando o provedor de identidade. 

- [Criar um app](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-register-applications?tabs=app-reg-ga)

Configuração|Valor
-|-
Name|b2c-workshop-app
Supported account types|Accounts in any identity provider or organizational directory (for authenticating users with user flows)
Redirect URI (recommended)|Web - https://jwt.ms
Authentication|Access tokens (used for implicit flows) ON
> Após criado, adicionar um _client secret_ em **Client & Secrets**


### Requisitando um token de aplicação
```bash
curl --location --request POST "https://login.microsoftonline.com/$TENANT/oauth2/v2.0/token" \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode "client_id=$CLIENT_ID" \
--data-urlencode "client_secret=$CLIENT_SECRET" \
--data-urlencode "grant_type=client_credentials" \
--data-urlencode "scope=.default"
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

> Crie um usuário e tente fazer o _login_ novamente.

## Adicionando uma nova claim
### User Attributes
Configuração|Valor
-|-
Name|Subscribe to Newsletter
Data Type|Boolean


## Adicionando o user flow de edição
Configuração|Valor
-|-
Profile editing|Recommended
Name|WORKSHOP_PROFILEEDIT
Local Accounts|Email signup
User attributes and token claims| Collect attribute e Return claim: Display Name e Subscribe to Newsletter

## Alterando o tipo do campo Subscribe to Newsletter
Page Layouts > Profile edit page > Subscribe to Newslleter 
Configuração|Valor
-|-
User Input Type|RadioSingleSelect
Text:yes|true
Text:no|false

## Criando uma Custom Policy
### Preparando o ambiente
- [Download e configuração da extensão do VS Code](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c)
- [Download do Starter Pack](
https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-user-flows?pivots=b2c-custom-policy#get-the-starter-pack)
- [Configurando as aplicações básicas para o uso de Custom Policies](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tutorial-create-user-flows?pivots=b2c-custom-policy#register-identity-experience-framework-applications).



### Preparando os arquivos do B2C
- Faça um Find/Replace em todos os arquivos, substituindo ```youtenant.onmicrosoft.com``` por ```{Settings:Tenant}``` 
- Em ```TrustFrameworkExtensions.xml``` substitua ```ProxyIdentityExperienceFrameworkAppId``` por ```{Settings:ProxyIdentityExperienceFrameworkAppId}```
- Em ```TrustFrameworkExtensions.xml``` substitua ```IdentityExperienceFrameworkAppId``` por ```{Settings:IdentityExperienceFrameworkAppId}```

Isso fará com que os valores em parenteses sejam substituídos pelos valores que estão no ```appsettings.json```

- Altere os valores do arquivo ```appsettings.json``` para os valores que estão no seu _tenant_ B2C
- Faça a compilação das policies utilizando o comando ```B2C Build All Policies```
- Vá em **Identity Framework Experience**:
    Vá em **Policy Keys** e crie uma as chaves:
    Configuração|Valor
    -|-
    Name|B2C_1A_TokenSigningKeyContainer
    Key Type|RSA
    Key Usage|Signature

    Configuração|Valor
    -|-
    Name|B2C_1A_TokenEncryptionKeyContainer
    Key Type|RSA
    Key Usage|Encryption

Faça o upload dos arquivos na seguinte ordem:
1. TrustFrameworkBase.xml
2. TrustFrameworkLocalization.xml
3. TrustFrameworkExtensions.xml
4. Outros 
    
Selecione a policy ```B2C_1A_SIGNUP_SIGNIN``` e faça o processo de SignUp/SignIn
