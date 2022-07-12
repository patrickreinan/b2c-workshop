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
**User Flows**: modelo que permite a criação de interfaces com o usuário de forma fácil, porém menos flexível.

**Custom Policies**: modelo baseado em configuração declarativa (XML) que permite customizar os fluxos de interação com o usuário
- [Visão Geral de User Flows e Custom Policies](
https://docs.microsoft.com/en-us/azure/active-directory-b2c/user-flow-overview?WT.mc_id=Portal-Microsoft_AAD_B2CAdmin)

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
- [Configure uma conta no Pipedream](https://pipedream.com/)


### Preparando os arquivos do B2C
- Faça um Find/Replace em todos os arquivos, substituindo ```yourtenant.onmicrosoft.com``` por ```{Settings:Tenant}``` 
- Em ```TrustFrameworkExtensions.xml``` substitua ```ProxyIdentityExperienceFrameworkAppId``` por ```{Settings:ProxyIdentityExperienceFrameworkAppId}```
- Em ```TrustFrameworkExtensions.xml``` substitua ```IdentityExperienceFrameworkAppId``` por ```{Settings:IdentityExperienceFrameworkAppId}```

Isso fará com que os valores em parenteses sejam substituídos pelos valores que estão no ```appsettings.json```

- Altere os valores do arquivo ```appsettings.json``` para os valores que estão no seu _tenant_ B2C
- Faça a compilação das policies utilizando o comando ```B2C Build All Policies```
- Na Azure Vá em **Identity Framework Experience**:
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

## Criando uma custom claim    
* [Claims customizadas no B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/user-flow-custom-attributes?pivots=b2c-custom-policy)


### Preparando o ambiente
* Localize o ```TechnicalProfile AAD-Common``` e atribua o ```ClientId``` e ```ApplicationObjectId``` da aplicação ```b2c-extensions-app. Do not modify. Used by AADB2C for storing user data.```.

```xml
 <ClaimsProvider>
    <DisplayName>Azure Active Directory</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="AAD-Common">
        <Metadata>
          <!--Insert b2c-extensions-app application ID here, for example: 11111111-1111-1111-1111-111111111111-->  
          <Item Key="ClientId"></Item>
          <!--Insert b2c-extensions-app application ObjectId here, for example: 22222222-2222-2222-2222-222222222222-->
          <Item Key="ApplicationObjectId"></Item>
        </Metadata>
      </TechnicalProfile>
    </TechnicalProfiles> 
  </ClaimsProvider>
```

- No ```TrustFrameworkBase.xml``` dentro de um ```ClaimSchema``` adicione a claim ```extension_subscribetonewsletter```.
```xml
<ClaimType Id="extension_subscribetonewsletter">
    <DisplayName>Subscribe to Newsletter</DisplayName>
    <DataType>boolean</DataType>
    <UserInputType>RadioSingleSelect</UserInputType>
        <Restriction>
        <Enumeration Text="Yes" Value="true" SelectByDefault="false" />
        <Enumeration Text="No" Value="false" SelectByDefault="false" />
        </Restriction>
</ClaimType>
```

Localize o ```Technical Profile SelfAsserted-ProfileUpdate``` e adicione a ```claim``` em ```InputClaims``` e ```OutputClaims```.

```xml
<InputClaims>
...
<InputClaim ClaimTypeReferenceId="extension_subscribetonewsletter" />
</InputClaims>

<OutputClaims>
...
  <OutputClaim ClaimTypeReferenceId="extension_subscribetonewsletter" />
</OutputClaims>
```
Localize o ```Technical Profile AAD-UserWriteProfileUsingObjectId```  adicione:

```xml
<PersistedClaim ClaimTypeReferenceId="extension_subscribetonewsletter" />
```
No arquivo ```ProfileEdit.xml``` adicione a ```outputClaim extension_subscribetonewsletter```
```xml
<OutputClaim ClaimTypeReferenceId="extension_subscribetonewsletter" PartnerClaimType="subscribetonewsletter"/>
```
Faça o build e upload dos arquivos TrustFrameworkBase.xml e ProfileEdit.xml e teste o fluxo de de edição de perfil
## Adicionando uma chamada REST

No arquivo ```TrustFrameworkBase.xml``` adicione a seção abaixo ao ```ClaimsProviders```

```xml

    <ClaimsProvider>
      <DisplayName>REST Claims Provider</DisplayName>
        <TechnicalProfiles>
          <TechnicalProfile Id="REST-GetRolesAndGroups">
            <DisplayName>Get Roles and Groups</DisplayName>
            <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
            <Metadata>
              <Item Key="ServiceUrl">https://enz9idfu8r7vg3.m.pipedream.net</Item>
              <Item Key="AuthenticationType">None</Item>
              <Item Key="SendClaimsIn">Header</Item>
              <Item Key="AllowInsecureAuthInProduction">True</Item>
            </Metadata>
            <InputClaims>
              <InputClaim ClaimTypeReferenceId="objectId" />
            </InputClaims>
            <OutputClaims>
              <OutputClaim ClaimTypeReferenceId="roles"  />
              <OutputClaim ClaimTypeReferenceId="groups"  />
            </OutputClaims>
            <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
          </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    


```

- No ```TrustFrameworkBase.xml``` dentro de ```BuildingBlocks\ClaimsSchema``` adicione:
```xml

  <ClaimType Id="groups">
      <DisplayName>Groups</DisplayName>
      <DataType>stringCollection</DataType>
  </ClaimType>

  <ClaimType Id="roles">
      <DisplayName>roles</DisplayName>
      <DataType>stringCollection</DataType>
  </ClaimType>

```

- No ```TrustFrameworkBase.xml``` na User Journey SignUpOrSignIn adicione o ```OrchestrationStep``` antes do último:
```xml
<OrchestrationStep Order="4" Type="ClaimsExchange">
  <ClaimsExchanges>
    <ClaimsExchange Id="RESTGetRoles" TechnicalProfileReferenceId="REST-GetRolesAndGroups" />
  </ClaimsExchanges>
</OrchestrationStep>
```
- Use o comando do Visual Studio Code ```B2C Renumber Policy```.

- No arquivo ```SignUpOrSignin.xml``` adicione as ```OutputClaims```.
```xml
<OutputClaim ClaimTypeReferenceId="roles"  />
<OutputClaim ClaimTypeReferenceId="groups"  />
```

Faça o procedimento de SignIn e verifique que as ```roles``` e ```groups``` estarão presentes no _token_.