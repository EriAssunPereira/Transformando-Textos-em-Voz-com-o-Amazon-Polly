# Transformando-Textos-em-Voz-com-o-Amazon-Polly

Para realizar a síntese de voz a partir de texto com o Amazon Polly, vamos usar e configurar o projeto utilizando AWS Lambda para integrar com o serviço Polly. Eu vou estruturar esse projeto em módulos.

### Passo 1: Configuração Inicial

1. **Configuração da AWS:**
   - Certifique-se de ter uma conta AWS e acesso à AWS Management Console.
   - Instale o AWS CLI e configure suas credenciais locais.

2. **Instalação do Terraform (opcional):**
   - Caso deseje automatizar a criação de recursos da AWS, instale o Terraform conforme mencionado anteriormente.

### Passo 2: Criação da Função Lambda

Vamos criar uma função Lambda que irá utilizar o serviço Amazon Polly para transformar texto em voz.

**`lambda_function.tf`**

```hcl
resource "aws_lambda_function" "text_to_speech" {
  filename      = "text_to_speech.zip"
  function_name = "TextToSpeechFunction"
  role          = aws_iam_role.lambda_role.arn
  handler       = "index.handler"
  runtime       = "nodejs14.x"

  environment {
    variables = {
      AWS_REGION   = var.aws_region
      POLLY_VOICE  = var.polly_voice
    }
  }
}
```

**`variables.tf`**

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
}

variable "polly_voice" {
  description = "Voice ID for Polly"
  type        = string
}
```

### Passo 3: Implementação da Lógica da Função Lambda

Implemente a lógica da função Lambda em Node.js para invocar o serviço Polly.

**`index.js`**

```javascript
const AWS = require('aws-sdk');
const polly = new AWS.Polly();

exports.handler = async (event) => {
  const text = event.text;
  
  const params = {
    OutputFormat: 'mp3',
    Text: text,
    TextType: 'text',
    VoiceId: process.env.POLLY_VOICE
  };
  
  try {
    const data = await polly.synthesizeSpeech(params).promise();
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'audio/mpeg',
        'Content-Length': data.AudioStream.length.toString(),
        'Cache-Control': 'no-cache, no-store, must-revalidate',
        'Pragma': 'no-cache',
        'Expires': '0'
      },
      body: data.AudioStream.toString('base64'),
      isBase64Encoded: true
    };
  } catch (err) {
    console.error('Error synthesizing speech:', err);
    return {
      statusCode: 500,
      body: JSON.stringify({ message: 'Error synthesizing speech' })
    };
  }
};
```

### Passo 4: Permissões e Roles

Configure as permissões necessárias para a função Lambda acessar o serviço Polly.

**`iam_role.tf`**

```hcl
resource "aws_iam_role" "lambda_role" {
  name = "lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Effect    = "Allow",
      Principal = {
        Service = "lambda.amazonaws.com"
      },
      Action    = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_policy_attachment" "lambda_polly_attach" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonPollyFullAccess"
  role       = aws_iam_role.lambda_role.name
}
```

### Passo 5: Implantação e Teste

1. **Empacote o Código da Função Lambda:**
   - Crie um arquivo `text_to_speech.zip` que contém `index.js` e quaisquer outras dependências necessárias.

2. **Inicialize o Terraform:**
   ```bash
   terraform init
   ```

3. **Planeje as Mudanças:**
   ```bash
   terraform plan
   ```

4. **Aplique as Mudanças:**
   ```bash
   terraform apply
   ```

### Passo 6: Utilização

- Após a implantação, você pode invocar a função Lambda via API Gateway ou diretamente para converter texto em voz utilizando Amazon Polly.
- Ajuste a variável `polly_voice` conforme a voz desejada disponível no Amazon Polly.

### Observações Finais

Certifique-se de ajustar e expandir este exemplo conforme suas necessidades específicas, como tratamento de erros, controle de versão do código da função Lambda, e segurança adequada para acessar serviços AWS. Este projeto oferece uma base sólida para explorar a síntese de voz a partir de texto utilizando o Amazon Polly na AWS de maneira automatizada e escalável.
