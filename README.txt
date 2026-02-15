🚀 Smart Dev-Docs Platform (SDDP)Smart Dev-Docs Platform là nền tảng xử lý tài liệu kỹ thuật thời gian thực. Hệ thống tự động hóa luồng dữ liệu từ khi tiếp nhận (Streaming), xử lý (ETL), lưu trữ (Lakehouse) cho đến khi suy luận AI (RAG-ready).🏗️ Kiến trúc Hệ thống (Data Flow)Ingestion: Dữ liệu JSON được đẩy vào Kafka (Topic: document-events).Processing: Spark Streaming tiêu thụ dữ liệu, thực hiện transform và ghi vào bảng.Storage: Apache Iceberg (với Hive Metastore) quản lý dữ liệu dưới dạng Table format, đảm bảo tính ACID.AI Layer: Ollama cung cấp các Model (Mistral, Nomic-Embed) để tạo vector embeddings và phản hồi thông minh.🛠️ Hướng dẫn Triển khai nhanh1. Khởi tạo ClusterĐảm bảo bạn đã cài đặt Minikube và kubectl.Bash# Triển khai toàn bộ stack (Kafka, Spark, Iceberg, Ollama, Postgres)
kubectl apply -f config/systems.yml

# Chờ các Pods chuyển sang trạng thái Running
watch kubectl get pods -A
2. Thiết lập Kafka TopicBash# Tạo topic cho tài liệu
kubectl exec -n kafka-kraft kafka-0 -- kafka-topics \
  --bootstrap-server localhost:9092 \
  --create --topic document-events --partitions 1 --replication-factor 1
📊 Thao tác với Dữ liệu (DML/DQL)Gửi dữ liệu mẫuSử dụng Console Producer để giả lập luồng streaming:Bashkubectl exec -n kafka-kraft -it kafka-0 -- kafka-console-producer \
  --bootstrap-server localhost:9092 \
  --topic document-events
Dán nội dung JSON mẫu vào terminal (xem chi tiết trong file gốc).Truy vấn Iceberg TableSử dụng Spark SQL để kiểm tra dữ liệu đã được ghi:Bashkubectl exec -n spark -it deployment/spark-streaming -- /opt/spark/bin/spark-sql \
  --conf spark.sql.catalog.hive_prod=org.apache.iceberg.spark.SparkCatalog \
  --conf spark.sql.catalog.hive_prod.type=hive \
  --conf spark.sql.catalog.hive_prod.uri=thrift://metastore-service.hive:9083 \
  -e "SELECT * FROM hive_prod.db.doc_chunks LIMIT 10;"
🤖 Tích hợp AI với OllamaHệ thống hỗ trợ RAG (Retrieval-Augmented Generation) thông qua Ollama API.Tác vụEndpointModel khuyến nghịEmbeddings/api/embeddingsnomic-embed-textChat/Inference/api/generatemistral:7b-instructVí dụ tạo Vector Embedding cho một đoạn văn bản:Bashcurl http://localhost:11434/api/embeddings -d '{
  "model": "nomic-embed-text",
  "prompt": "Apache Iceberg is a high-performance format for huge analytic tables"
}'
🔍 Giám sát & LogsSpark UI: kubectl port-forward -n spark svc/spark-ui 4040:4040 -> Truy cập localhost:4040Logs Spark: kubectl logs -f -n spark deployment/spark-streamingLogs Ollama: kubectl logs -f -n ollama deployment/ollama⚠️ Lưu ý kỹ thuật (Troubleshooting)[!IMPORTANT]Ivy Cache: Nếu Spark không thể tải jar, hãy kiểm tra quyền ghi tại thư mục /tmp/.ivy2.Ollama Models: Nếu Model chưa có sẵn, hãy exec vào pod và chạy: ollama pull mistral.
