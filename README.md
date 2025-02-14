# High-Performance Content Rating API

An optimized **Django REST Framework (DRF)**-based application designed to manage user ratings and content interactions. This system efficiently processes large-scale ratings while minimizing fraudulent or biased scores. Performance optimizations ensure it can handle thousands of requests per second seamlessly.

---

## Key Capabilities

### 1. Retrieve Content Listings
- Displays all content along with:
  - **Name**
  - **Total Ratings Count**
  - **Mean Rating Score**
- Optimized using precomputed data and optimized database queries for speed.
- **Pagination** ensures smooth handling of extensive datasets without performance bottlenecks.
- **Caching Mechanism**: The content list is cached for a duration of five minutes, reducing database queries. If cache data expires, the system fetches fresh data and stores it again.

### 2. Submit and Update Ratings
- Users can assign a rating between **0 and 5** to any content.
- If a user has previously rated an item, their score is updated instead of duplicated.
- High-efficiency architecture to manage millions of ratings without slowdown.
- **Indexes** on frequently accessed fields (`user_id`, `content_id`, and `title`) improve query performance.
- **Cache Storage**: Each user's rating for content is cached for one hour, cutting down redundant database reads.

#### API Endpoints for Rating and Content Interaction

##### **Content Management Endpoint (`ContentView`)**:
Provides an optimized paginated listing of content with caching.

- **GET Request**: Retrieves a paginated list of content displaying:
  - **Title**
  - **Total ratings**
  - **Mean score**
  
- **Optimizations**:
  - **Pagination Enabled**: Default page size is 100 for efficient data retrieval.
  - **Cache Utilization**: Content list is cached for 5 minutes (`all_contents`) to limit database queries. If cache is expired or unavailable, data is retrieved from the database and stored again.

##### **Rating Submission Endpoint (`ScoreView`)**:
Allows users to submit or modify their ratings for content, leveraging performance enhancements.

- **POST Request**: Accepts rating submissions.
  - If a user has already rated an item, the rating is updated.
  - If no prior rating exists, a new entry is recorded.

- **Optimizations**:
  - **Atomic Database Transactions**: Ensures consistency using `transaction.atomic()`.
  - **Cached Ratings**: Ratings per user-content pair are stored for 1 hour, reducing database dependency.
  - **Optimized Updates**: Unchanged ratings are not unnecessarily updated in the database, reducing write operations.
  - **Real-Time Score Updates**: Mean ratings are dynamically adjusted upon new entries or modifications.
  - **Concurrency Handling**: Implements `select_for_update()` to avoid simultaneous rating updates causing conflicts.

### 3. Fraud Prevention Mechanism
- Detects abnormal rating behaviors by analyzing the past hour's ratings against the overall trend.
- If ratings from the last hour deviate beyond **two standard deviations** from the mean score, and sufficient ratings exist, the system flags them as suspicious and adjusts calculations accordingly.
- **Background Task Processing**: Celery and Redis are used for asynchronous fraud detection, preventing performance drops during real-time operations.
- **Advanced Detection Possibility**: More sophisticated machine learning techniques, such as **Random Forest**, could be integrated in the future to enhance fraud identification beyond basic statistical checks.

---

## Installation Guide

### Requirements
- Python 3.9+
- Django 4.0+
- PostgreSQL (recommended for production use)
- Redis (for handling background task queues)

### Setup Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/content-rating-api.git
   cd content-rating-api
   ```
2. Install required dependencies:
    ```bash
    pip install -r requirements.txt
    ```
3. Configure PostgreSQL and Redis as required.
4. Apply database migrations:
    ```bash
    python manage.py migrate
    ```
5. (Optional) Populate with sample data:
   Use `populate_data.py` to generate test data (1000 users & 100 content items with ratings):
    ```bash
    python populate_data.py
    ```

---

## Optimization Techniques

### 1. **Efficient Load Testing (`send_requests.py`)**
- Utilizes `asyncio` and `aiohttp` to send multiple requests concurrently.
- Implements concurrency control using a **Semaphore**, preventing server overload.
- Performance Gains:
  - Before optimization: Processing **200 requests took ~50 seconds**.
  - After optimization: The same **200 requests now execute in ~10 seconds**.

### 2. **Database and Cache Enhancements**
- Frequently accessed queries leverage **caching** for improved response time.
- **Indexes** on critical fields reduce query execution time significantly.

### 3. **Asynchronous Fraud Detection via Celery**
- Fraudulent ratings are identified in real-time using background tasks.
- This approach ensures fraudulent patterns don’t affect rating integrity while keeping the API responsive.
- The system flags ratings deviating significantly from expected trends as potential fraud.

### 4. **Performance Improvements Summary**
**Before Enhancements:**
- Processing 200 requests: **~50 seconds**

**After Optimizations:**
- Processing 200 requests: **~10 seconds**

---

## Future Enhancements
- Enhance fraud detection with **machine learning models** like Random Forest to analyze complex rating behaviors.
- Further optimize caching strategies for enhanced scalability.
- Improve user experience with WebSocket support for real-time rating updates.

---

## Conclusion
This content rating API is engineered for **scalability, performance, and reliability**. By integrating caching, optimized queries, and fraud prevention, the system ensures accurate ratings while handling significant loads efficiently. It is production-ready and can be expanded for future improvements.

