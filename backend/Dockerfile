FROM node:16-alpine
WORKDIR /app

# Install dependencies
COPY backend/package*.json ./
RUN npm install --only=production

# Copy backend source code
COPY backend/. .

EXPOSE 3000
CMD ["npm", "start"]
