# **Lambda@Edge vs CloudFront Functions**

## **Overview**
- Both run **at CloudFront edge locations** globally
- **No servers to manage** - fully serverless
- **Pay-per-use** model
- Enable **customization close to users** for lower latency

---

## **CloudFront Functions**
**Purpose**: Ultra-fast, simple request/response modifications
- **Runtime**: JavaScript only
- **Execution time**: <1ms
- **Scale**: Millions of requests/sec
- **Trigger points**: Only **viewer request & viewer response**
- **Features**:
  - No network access
  - No filesystem access
  - Can't use AWS SDK
  - Very lightweight

**Use Cases**:
- Cache key normalization
- Header manipulation (add/modify/remove)
- URL rewrites/redirects
- JWT token validation (simple)
- A/B testing (basic)

---

## **Lambda@Edge**
**Purpose**: Complex logic with longer execution
- **Runtimes**: Node.js & Python
- **Execution time**: 5-30 seconds (depending on region)
- **Scale**: Thousands of requests/sec
- **Trigger points**: **All 4**:
  1. Viewer request
  2. Origin request
  3. Origin response
  4. Viewer response
- **Features**:
  - Network access (external APIs)
  - Filesystem access (500MB temp space)
  - Can use AWS SDK
  - CPU/memory configurable

**Use Cases**:
- Bot mitigation
- Dynamic content generation
- Real-time image transformation
- Complex authentication/authorization
- SEO optimization
- Origin failover/routing
- Response compression/formatting

---

## **Key Differences Summary**
| **Aspect** | **CloudFront Functions** | **Lambda@Edge** |
|------------|-------------------------|-----------------|
| **Runtime** | JavaScript only | Node.js, Python |
| **Max Duration** | <1ms | 5-30 seconds |
| **Scale** | Millions/sec | Thousands/sec |
| **Cost** | Very cheap | More expensive |
| **Complexity** | Simple | Complex logic |
| **SDK Access** | No | Yes |
| **Network** | No | Yes |
| **Deployment** | CloudFront UI/API | us-east-1 only |

---

## **When to Use Which?**

### **Choose CloudFront Functions when:**
- Need **microsecond** response times
- Doing **simple** header/URL manipulation
- Handling **massive** scale (millions RPS)
- Cost is **critical**

### **Choose Lambda@Edge when:**
- Need **complex logic** or external API calls
- Require **AWS SDK** integration
- Processing takes **>1ms**
- Need to modify **origin request/response**

---

## **Deployment Note**
- **CloudFront Functions**: Deploy directly to CloudFront
- **Lambda@Edge**: Deploy in **us-east-1** → auto-replicated globally
- Both: Attach to **CloudFront distribution behaviors**