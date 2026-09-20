# Loopcv questions

### 1. Multi-language error handling

Our platform supports multiple languages. When the backend API returns an error, we need to show a translated, user-friendly message on the frontend — regardless of the user's selected language.

How would you design this? Specifically:

- What should the API error response look like?
- How does the frontend map that to the correct translated message?
- How do you handle unexpected/unknown errors that don't have a predefined translation?

A quick example (response shape or code snippet) is welcome but not required.

#### Answer

The proper architectural approach for managing multiple languages is to handle it exclusively on the frontend. The backend's responsibility is solely to return all necessary context about the error, while the frontend dictates how that information is presented to the user based on their locale.

##### 1. What should the API error response look like?

At the backend level, API error response should follow an established structure that is consistent across all requests.

``` json
{
  "error": {
    "code": "AUTH_INCORRECT_PASSWORD",
    "message": "The password provided does not match our records.",
    "details": {
      "attemptsRemaining": 2
    },
    "traceId": "req_9f82c401"
  }
}
```
Field breakdown:

- `code`: An enum that we will use to display the appropriate localized text to the user.
- `message`: An English description intended strictly for developer debugging. We should never display this to user.
- `details`: An optional object containing dynamic values. This allows the frontend to interpolate specific variables directly into the translated string.
- `traceId`: A unique request identifier used to track the error within our backend logs and monitoring systems.

##### 2. How does the frontend map that to the correct translated message?

Since the platform already supports multiple languages, we should leverage the existing internationalization library (e.g., `vue-i18n`).

In our locales directory, we will add an `errors.js` file for each language. In these files, we will map the defined error codes to their proper translation texts. The structure must remain identical across all language directories to ensure consistency and prevent missing translations.

``` javascript
// locales/en/errors.js
export default {
  errors: {
    AUTH_INCORRECT_PASSWORD: "The password you entered is incorrect. You have {attemptsRemaining} attempts left.",
    GENERIC_ERROR: "An unexpected error occurred. Please try again later."
  }
};

// locales/el/errors.js
export default {
  errors: {
    AUTH_INCORRECT_PASSWORD: "Ο κωδικός πρόσβασης είναι λανθασμένος. Έχετε {attemptsRemaining} προσπάθειες ακόμα.",
    GENERIC_ERROR: "Παρουσιάστηκε σφάλμα. Παρακαλώ δοκιμάστε ξανά αργότερα."
  }
};
```

Then, we will use a utility function to properly get the correct translated messages when needed.

``` javascript
import { i18n } from '@/i18n';

/**
 * Parses a standardized API error response and returns the localized string.
 */
export function getLocalizedErrorMessage(apiPayload) {
  const errCode = apiPayload?.error?.code;
  const details = apiPayload?.error?.details || {};
  
  // Check if we have a valid code AND if it exists in our dictionary
  if (errCode && i18n.global.te(`errors.${errCode}`)) {
    // Interpolate the 'details' object into the translated string
    return i18n.global.t(`errors.${errCode}`, details);
  }

  // Fallback for unknown errors
  return i18n.global.t('errors.GENERIC_ERROR');
}
```

##### 3. How do you handle unexpected/unknown errors that don't have a predefined translation?

In the `getLocalizedErrorMessage()` function, we return a fallback error message (`errors.GENERIC_ERROR`) in case the error code is undefined, missing from the payload, or doesn't exist in our translation dictionary.

This guarantees that the user always sees a localized message in the event of an unhandled error.

---

### 3. Handling an important client's task

An important client reports an issue/task that needs to be handled. Walk me through how you'd approach it from start to finish — and how would you decide it's actually "done"?

#### Answer

Initially, we examine the reported problem to fully understand it and identify exactly where in the application (functionality, feature, or page) it occurs. If we need clarification or additional information, we reach out to the client.

Next, we attempt to reproduce the problem. If successful, we analyze the issue by asking the following questions:

- Is the application's current behavior or output actually correct?
- Should the client have access to this specific feature or page?
- What is the expected behavior or outcome?

Based on the answers, we determine our approach:

- **If the behavior is correct**: We consider whether UI/UX improvements are needed to make the intended functionality clearer to the client.
- **If the client should not have access**: We audit our implementation to locate where user permissions are failing and fix the missing checks.
- **If the behavior is incorrect**: We trace the data flow and execution steps to identify the root cause. A key part of this step is isolating exactly where the problem originates, whether it is a frontend issue, a backend issue, or both. When we successfully identify the issue we then proceed to implement the necessary fixes.

If the issue can't be reliably reproduced or traced, the process becomes more challenging. In such cases, we need to make assumptions about what might be going wrong and attempt to resolve the issue using the steps above.

After implementing the necessary changes, we perform the following checks:

- We conduct manual testing and write new automated tests (if the project supports it) to cover all edge cases and prevent regressions.
- We investigate whether the issue exists in other parts of the application that share this functionality and apply the fix globally if necessary.
- We verify that our updates do not negatively impact performance or introduce new bugs into other areas of the application.

Once all changes and verifications are complete, we follow the company's standard deployment pipeline (code review, deployment and QA in the staging environment, deployment and QA in production).

After verifying the resolution in production, we notify the client and provide all relevant details. In all cases, an issue is only considered "done" once we receive confirmation from the client that everything is working properly.
