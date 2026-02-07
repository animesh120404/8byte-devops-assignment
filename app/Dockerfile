# STAGE 1: Builder
FROM node:18-alpine AS builder

WORKDIR /app
    
# Copy package files first to leverage Docker cache
COPY package*.json ./
    
# Install all dependencies
RUN npm ci
    
# Copy the rest of the application code
COPY . .
    
# STAGE 2: Runner
FROM node:18-alpine AS runner
    
# Set environment to production
ENV NODE_ENV=production
    
WORKDIR /app

# Only copy necessary files from the builder stage
COPY --from=builder --chown=node:node /app/package.json ./
COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --from=builder --chown=node:node /app/app.js ./
    
# Documentation for the port the app listens on
EXPOSE 3000
    
# Run as a non-root user for security
USER node

CMD ["node", "app.js"]