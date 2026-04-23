# --- Frontend build stage ---
FROM node:16.16.0 AS frontend-build
WORKDIR /app

COPY package*.json ./
RUN npm ci || npm install

COPY . .
RUN npm run build


# --- Backend runtime stage ---
FROM ubi8/python-311
WORKDIR /app

# OpenShift compatibility: random UID, group-writable dir
RUN chmod -R g+rwx /app

# Copy backend + built frontend
COPY server ./server
COPY run.py ./run.py
COPY --from=frontend-build /app/dist ./dist

# Install Python deps
COPY requirements.txt ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

ENV HOST=0.0.0.0
ENV PORT=8080

EXPOSE 8080

# Run backend
CMD ["python", "run.py"]