# Instaladores Jovin

O workflow `Jovin installers` gera instaladores QZ Tray provisionados para Windows x64 e Linux x64
sem versionar certificados. Ele só pode ser iniciado manualmente por alguém com permissão de escrita
no repositório e usa, por padrão, a versão oficial `v2.3.0`.

A matriz completa herdada do projeto upstream está preservada como `upstream build`, disponível para
execução manual. Ela não roda a cada push ou pull request do fork para evitar consumo desnecessário
de runners Windows e macOS.

## Configuração única no GitHub

Cadastre em **Settings → Secrets and variables → Actions → New repository secret**:

- `JOVIN_QZ_CA_CRT_BASE64`: conteúdo Base64 do certificado público da autoridade Jovin usada para
  validar as assinaturas de impressão.

Em Linux, gere o valor sem quebras de linha:

```bash
base64 -w 0 jovin-qz-ca.crt
```

O certificado público é material de confiança, não uma chave secreta, mas fica no GitHub Actions
para que nenhum certificado seja gravado no código ou no histórico do fork. A chave privada
correspondente nunca deve ser cadastrada neste repositório, no Actions ou nos instaladores; ela fica
somente no backend Jovin.

## Geração e publicação

Abra **Actions → Jovin installers → Run workflow**, confira a origem HTTPS do sistema e informe uma
tag de distribuição própria, por exemplo `jovin-qz-v2.3.0.1`. O job:

1. valida os parâmetros;
2. baixa o código fixado do QZ Tray;
3. decodifica o certificado apenas no diretório temporário do runner;
4. gera o provisionamento para a origem Jovin;
5. monta os pacotes `.exe` e `.run`;
6. calcula `SHA256SUMS`;
7. publica os arquivos como artifact e, quando habilitado, em uma GitHub Release.

Configure as URLs públicas da Release no backend usando `PRINT_INSTALLER_WINDOWS_URL` e
`PRINT_INSTALLER_LINUX_URL`. O executável Windows não recebe assinatura Authenticode neste fluxo;
até uma assinatura de código ser configurada, o Windows poderá exibir aviso de editor desconhecido.
