# Instalação

Utilizou-se NVM para gerenciar a versão do nodejs (10.16.3).

- Dependências de terceiros

  - Instalar as seguintes dependencias:

  ```bash
  sudo apt install git redis-server r-base cloc
  ```

  - Instalar o [MOngodb](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/#std-label-install-mdb-community-ubuntu)

- Instalação [NodeRacer](https://github.com/andreendo/noderacer/blob/master/README.md?plain=1)

  - Siga os passos:
    obs: Lembre-se de utilizar a versão 10.16.3

  ```bash
  git clone https://github.com/andreendo/noderacer.git
  cd noderacer
  npm install
  npm link
  ```

#
