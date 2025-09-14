
## Basic Questions

### Logic Design

1. A user uploads a photo through the client application.
2. The backend API receives the file, uploads it directly to a private S3 bucket, and creates a new record in the `photos` database table with an initial status of `pending_automated_review`.
3. The manual review API queries the database for the list of photos in the `pending_manual_review` state, presenting them in a queue for       human moderators to make the final decision.
4. Once the moderators approve, the status of the photo will be changed to `approved`
5. The published photos API will filter the photos to only shows `approved` photos

### Technologies Used
1. **API Gateway & Load Balancer** for a secure and scalable entry point for all incoming photo upload requests
2. **Object Storage (e.g., AWS S3):** Provides durable, scalable, and cost-effective storage for the raw image files.
3. **Database (e.g., PostgreSQL on AWS RDS):** A relational database to store structured metadata for each photo, including its storage location, ownership, current moderation status, and a full history of actions taken
4. **Golang** for backend service language, providing a scalable backend service for photo upload application
5. **React/Vue/Angular with Typescript** for frontend service framework, providing scalable and maintainable frontend service for the interface application for users to upload the photos and see the progress of the moderation and moderators for reviewing and approving the photos
### Data Structure
1. For the file itself, it will be stored as object in the object storage such as AWS S3'
2. For the metadata, it will have mainly 2 tables: `photos` for storing the photo metadata and `moderation_logs` for storing moderation history. Sample definition can be seen below:
```
    CREATE TABLE photos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    s3_bucket VARCHAR(255) NOT NULL,
    s3_key VARCHAR(1024) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'pending_automated_review', -- e.g., pending_automated_review, pending_manual_review, approved, rejected
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE moderation_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    photo_id UUID NOT NULL REFERENCES photos(id),
    moderator_id UUID, -- Can be NULL for automated actions
    action VARCHAR(50) NOT NULL, -- e.g., automated_flagged, manual_approved, manual_rejected
    notes TEXT, -- Comments from human moderators
    raw_moderation_data JSONB, -- Stores the raw JSON response from the AI service
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

## Database Questions
1. `SELECT COUNT(*) FROM Customers WHERE Country = 'Germany';`
2. `SELECT
    COUNT(CustomerID),
    Country
FROM
    Customers
GROUP BY
    Country
HAVING
    COUNT(CustomerID) >= 5
ORDER BY
    COUNT(CustomerID) DESC;`
3.    `SELECT
    c.CustomerName,
    COUNT(o.OrderID) AS OrderCount,
    MIN(o.OrderDate) AS FirstOrder,
    MAX(o.OrderDate) AS LastOrder
FROM
    Customers c
JOIN
    Orders o ON c.CustomerID = o.CustomerID
GROUP BY
    c.CustomerName
HAVING
    COUNT(o.OrderID) >= 5
ORDER BY
    LastOrder DESC
LIMIT 9;`

## JavaScript/TypeScript Questions
1. `function titleCase(str: string): string {
  if (!str) {
    return "";
  }
  return str
   .toLowerCase()
   .split(' ')
   .map(word => word.charAt(0).toUpperCase() + word.slice(1))
   .join(' ');
}`
2. `function delay(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}
// Usage:
delay(3000).then(() => alert('runs after 3 seconds'));`
3.
```
function fetchData(url: string): Promise<string> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!url) {
        return reject(new Error("URL is required"));
      }
      resolve(`Data from ${url}`);
    }, 1000);
  });
}

function processData(data: string): Promise<string> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!data) {
        return reject(new Error("Data is required"));
      }
      resolve(data.toUpperCase());
    }, 1000);
  });
}

// using the functions 
async function main() {
  try {
    const data = await fetchData("https://example.com");
    const processedData = await processData(data);
    console.log("Processed Data:", processedData);
  } catch (error) {
    console.error("An error occurred:", error);
  }
}

main();
```
3. The app can be accessed https://staging.d2n5dex9fqi5is.amplifyapp.com/
## Vue.js
### **Vue.js reactivity and common issues when tracking changes**
When a component's `data` is initialized, Vue wraps the object in a `Proxy`. This proxy intercepts property access.
The process works in two main phases:
1.  **Dependency Tracking:** When a component's render function is executed, it accesses properties of reactive objects (e.g., `state.message`). The `Proxy`'s `get` trap is triggered for each property access. Vue's reactivity system records that the component's render effect is a "subscriber" to that specific property. This creates a map of dependencies where each property knows which effects depend on it.

2.  **Change Notification:** When a property on the reactive object is modified (e.g., `state.message = 'New value'`), the `Proxy`'s `set` trap is triggered. The system then looks up all the effects (subscribers) that depend on this property and re-runs them. For a component, this means its render function is re-executed, updating the DOM with the new value.

**Common Issues:**
-   **Losing Reactivity on Destructuring:** A frequent mistake is destructuring a reactive object. For example, `const { count } = reactive({ count: 0 })` creates a local variable `count` that holds the initial value (0) but is disconnected from the underlying proxy. Modifying this variable will not trigger updates. The correct way to destructure while preserving reactivity is to use the `toRefs()` utility: `const { count } = toRefs(state)`.

-   **Adding New Properties (Vue 2 Legacy):** In Vue 2, which used `Object.defineProperty`, adding a new root-level property to a reactive object after its creation would not make it reactive. This limitation is a primary reason for the move to `Proxy` in Vue 3, which transparently handles the addition of new properties.

-   **Unintended Reactivity with External State:** When integrating large, external state objects (e.g., from a library), making the entire object deeply reactive can be inefficient. In these cases, using `shallowRef` or `shallowReactive` is a best practice. These utilities only make the top-level properties reactive, preventing Vue from traversing and converting a large, complex object graph.

### Describe data flow between components in a Vue.js app
he primary data flow pattern in Vue.js is a strict one-way data flow, commonly summarized as **"props down, events up"**.

-   **Props Down:** Data is passed from a parent component to a child component via `props`. Props are custom attributes on a component tag. The child component must explicitly declare the props it expects to receive. When the parent's state changes, these changes automatically flow down to the child through the reactive props.

-   **Events Up:** A child component should never directly mutate a prop it receives from its parent. This prevents child components from accidentally affecting the parent's state, which would make the application's data flow difficult to reason about. Instead, to communicate a change or action back to the parent, the child component emits a custom `event`. The parent component can then listen for this event and, in its handler, update its own state. This completes the communication loop while maintaining a predictable, unidirectional flow of data.


For more complex scenarios where this pattern becomes cumbersome, Vue provides other mechanisms:

-   **Provide/Inject:** Used for passing data from an ancestor component to any of its descendants, regardless of how deeply nested they are. This avoids "prop drilling" through many intermediate components.

-   **Centralized State Management (Pinia):** For application-wide state that needs to be accessed or modified by many disparate components, a centralized store like Pinia is the recommended solution. It acts as a single source of truth, and any component can read from or dispatch actions to the store.

### List the most common cause of memory leaks in Vue.js apps and how they can be solved.
1. Global Event Listeners not removed properly. Solution: Always remove the event listener in the `onUnmounted` lifecycle hook.
2. Timers and Intervals (`setTimeout`, `setInterval`) runs forever. Solution: Store the ID returned by `setInterval` and use `clearInterval` in the `onUnmounted` hook. The same applies to `setTimeout` with `clearTimeout`.
3. Subscriptions (WebSockets, RxJS, etc.). Solution: Store the subscription object and call its `unsubscribe()` method in the `onUnmounted` hook.
### What have you used for state management
Vuex
### What's the difference between pre-rendering and server-side rendering?
-   **erver-Side Rendering (SSR):** The HTML for a page is generated on the server **at request-time**. Every time a user requests a page, the Node.js server runs the Vue application, renders the appropriate components into an HTML string, and sends that back to the browser. This is ideal for pages with highly dynamic, user-specific, or frequently changing content.

-   **Pre-rendering (SSG):** The HTML for every page is generated **at build-time**. During the deployment process, the application is run, and the output for each route is saved as a static `.html` file. These files are then deployed to a static host or CDN. This approach is extremely fast and cost-effective but is only suitable for content that is the same for all users and does not change frequently between builds (e.g., blogs, marketing pages, documentation).

## Website Security Best Practises
1. Enforce Strict Access Control
2. Prevent Injection Flaws
3. Use Strong Cryptography
4. Secure Authentication and Session Management
5. Maintain Secure Configurations
6. Manage Vulnerable and Outdated Components
7. Prevent Server-Side Request Forgery (SSRF)
8. Ensure Software and Data Integrity
9. Implement Security Logging and Monitoring
10. Practice Secure Design

## Website Performance Best Practises
1. Backend & Database Performance
    - Efficient Database Queries
    - Implement Backend Caching
    - Use Asynchronous Workers
2. Network & Asset Delivery
    - Use a Content Delivery Network (CDN)
    - Enable Compression
    - Leverage Browser Caching
    - Minimize HTTP Requests
3. Frontend Rendering Performance
    - Optimize the Critical Rendering Path
    - Image Optimization
    - Lazy Loading
    - Code Splitting
    - Defer Non-critical JavaScript

## Golang
```
package main

import (
	"fmt"
	"regexp"
	"strings"
)

func WordFrequency(text string) map[string]int {
	lowerText := strings.ToLower(text)

	reg := regexp.MustCompile(`[^a-z0-9\s]+`)
	cleanedText := reg.ReplaceAllString(lowerText, "")

	words := strings.Fields(cleanedText)
	frequencies := make(map[string]int)

	for _, word := range words {
		frequencies[word]++
	}

	return frequencies
}

func main() {
	inputString := "Four, One two two three Three three four  four   four"

	wordCounts := WordFrequency(inputString)

	fmt.Println("Word frequencies for the string:")
	fmt.Printf("'%s'\n\n", inputString)
	for word, count := range wordCounts {
		fmt.Printf("%s => %d\n", word, count)
	}
}
```
## Tools (Rate yourself 1 to 5)
-   Git: 5
-   Redis: 4
-   VSCode / JetBrains: 4
-   Linux: 3
-   AWS: 3
-   EC2: 3
-   Lambda: 4
-   RDS: 4
-   Cloudwatch: 3
-   S3: 5
-   Unit testing: 5
-   Kanban boards: 5