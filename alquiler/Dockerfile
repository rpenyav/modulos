# Usar una imagen base de Node
FROM node:16

# Establecer el directorio de trabajo en el contenedor
WORKDIR /app

# Copiar el package.json y el package-lock.json
COPY package*.json ./

# Instalar dependencias
RUN npm install

# Copiar el resto de los archivos del directorio
COPY . .

# Ejecutar el comando de inicio (ajusta esto según tu configuración)
CMD [ "npm", "run", "serve:single-spa:alquiler" ]
