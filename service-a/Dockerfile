FROM nginx:latest

# Remove default nginx static files
RUN rm -rf /usr/share/nginx/html/*

# Copy your app files into nginx folder
COPY service-a /usr/share/nginx/html

# Expose port 80
EXPOSE 80