# المرحلة الأولى: البناء (Build Stage)
FROM golang:1.21-alpine AS builder

# تثبيت الاعتماديات اللازمة
RUN apk add --no-cache git

WORKDIR /app

# نسخ ملفات الوحدات (Go modules) لتسريع عملية البناء عند التغيير
COPY go.mod go.sum ./
RUN go mod download

# نسخ باقي شيفرة المصدر
COPY . .

# بناء التطبيق كملف تنفيذي ثابت (static executable)
# تعطيل CGO يقلل من حجم الصورة النهائية
RUN CGO_ENABLED=0 go build -o /blog-api main.go

# المرحلة الثانية: التشغيل (Run Stage)
FROM alpine:latest

# تثبيت الشهادات الجذرية (مطلوبة لبعض الاتصالات)
RUN apk --no-cache add ca-certificates

# التصحيح (1): إنشاء مسار ServiceAccount لحل مشكلة 'read-only file system'
RUN mkdir -p /var/run/secrets/kubernetes.io/serviceaccount

# نسخ الثنائي المُنشأ من المرحلة الأولى
COPY --from=builder /blog-api /usr/local/bin/

# *** التصحيح الحاسم (2): إضافة صلاحيات التنفيذ للثنائي ***
RUN chmod +x /usr/local/bin/blog-api

# تعيين نقطة الدخول
ENTRYPOINT ["/usr/local/bin/blog-api"]

# تعيين المنفذ الافتراضي (إذا كان تطبيق Go يستمع على 8000)
EXPOSE 8000
