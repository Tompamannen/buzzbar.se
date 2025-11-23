# Analytics Implementation Design Document

## Overview

This design document outlines the implementation of Google Analytics 4 (GA4) tracking across the Vero energy bar website. The solution will provide comprehensive tracking of user behavior, conversions, and engagement while maintaining performance and privacy compliance.

The implementation uses GA4's modern event-based tracking model with enhanced e-commerce tracking for the product purchase funnel. All tracking code will be added directly to HTML pages with a consistent pattern that can be easily maintained and updated.

## Architecture

### High-Level Architecture

```
┌─────────────────┐
│   User Browser  │
└────────┬────────┘
         │
         ├─── Page Load
         │
    ┌────▼─────────────────┐
    │   HTML Pages         │
    │  (index, Products,   │
    │   Purchase, etc.)    │
    └────┬─────────────────┘
         │
         ├─── Load gtag.js (async)
         │
    ┌────▼─────────────────┐
    │   Google Analytics   │
    │   Tracking Code      │
    │   (gtag.js)          │
    └────┬─────────────────┘
         │
         ├─── Send Events
         │
    ┌────▼─────────────────┐
    │   Google Analytics   │
    │   4 Platform         │
    │   (Cloud)            │
    └──────────────────────┘
```

### Component Structure

1. **Base Tracking Code**: Core GA4 snippet added to `<head>` of all pages
2. **Event Tracking Layer**: Custom event tracking for user interactions
3. **E-commerce Tracking**: Enhanced e-commerce events for purchase funnel
4. **Page-Specific Tracking**: Custom tracking for unique page features

## Components and Interfaces

### 1. Base Analytics Snippet

**Location**: `<head>` section of all HTML pages  
**Purpose**: Initialize GA4 and track page views

```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX', {
    'send_page_view': true,
    'anonymize_ip': true
  });
</script>
```

**Configuration Options**:
- `send_page_view`: Automatically track page views
- `anonymize_ip`: Enable IP anonymization for privacy compliance
- Async loading prevents blocking page render

### 2. CTA Button Tracking

**Location**: Inline event handlers or event listeners  
**Purpose**: Track engagement with call-to-action buttons

**Implementation Pattern**:
```javascript
// Track Shop Now clicks
function trackShopNow(location) {
  gtag('event', 'cta_click', {
    'event_category': 'engagement',
    'event_label': 'shop_now',
    'button_location': location
  });
}

// Track Learn More clicks
function trackLearnMore(destination) {
  gtag('event', 'cta_click', {
    'event_category': 'engagement',
    'event_label': 'learn_more',
    'destination_page': destination
  });
}
```

### 3. E-commerce Event Tracking

**Location**: Product selection and purchase pages  
**Purpose**: Track the complete purchase funnel

**Event Flow**:
```
Product View → Select Item → Begin Checkout → View Cart → Purchase
```

**Implementation**:

```javascript
// When user selects a product pack
function trackProductSelection(productName, price, quantity) {
  gtag('event', 'select_item', {
    'event_category': 'ecommerce',
    'items': [{
      'item_name': productName,
      'price': price,
      'quantity': quantity,
      'currency': 'USD'
    }]
  });
}

// When user clicks "Select Pack"
function trackBeginCheckout(productName, price, quantity) {
  gtag('event', 'begin_checkout', {
    'event_category': 'ecommerce',
    'value': price,
    'currency': 'USD',
    'items': [{
      'item_name': productName,
      'price': price,
      'quantity': quantity
    }]
  });
}

// On Purchase page load
function trackViewCart(productName, price, quantity) {
  gtag('event', 'view_cart', {
    'event_category': 'ecommerce',
    'value': price,
    'currency': 'USD',
    'items': [{
      'item_name': productName,
      'price': price,
      'quantity': quantity
    }]
  });
}

// When purchase is completed
function trackPurchase(transactionId, productName, price, quantity) {
  gtag('event', 'purchase', {
    'transaction_id': transactionId,
    'value': price,
    'currency': 'USD',
    'items': [{
      'item_name': productName,
      'price': price,
      'quantity': quantity
    }]
  });
}
```

### 4. Content Engagement Tracking

**Location**: Pages with interactive content  
**Purpose**: Track user engagement with specific content elements

**Implementation**:
```javascript
// Track carousel interactions
function trackCarouselNavigation(direction, currentSlide) {
  gtag('event', 'carousel_interaction', {
    'event_category': 'engagement',
    'event_label': direction,
    'slide_number': currentSlide
  });
}

// Track scroll depth
function trackScrollDepth(section, depth) {
  gtag('event', 'scroll', {
    'event_category': 'engagement',
    'event_label': section,
    'scroll_depth': depth
  });
}

// Track content expansion (FAQ, nutrition details)
function trackContentExpansion(contentType, contentTitle) {
  gtag('event', 'content_interaction', {
    'event_category': 'engagement',
    'event_label': contentType,
    'content_title': contentTitle
  });
}
```

## Data Models

### Event Data Structure

All custom events follow this structure:

```javascript
{
  event_name: string,           // e.g., 'cta_click', 'select_item'
  event_category: string,        // e.g., 'engagement', 'ecommerce'
  event_label: string,           // Specific action identifier
  // Additional custom parameters
  [key: string]: any
}
```

### E-commerce Item Structure

```javascript
{
  item_name: string,             // e.g., 'Starter Pack', 'Duo Pack'
  price: number,                 // e.g., 29, 54, 75
  quantity: number,              // Number of bars (12, 24, 36)
  currency: 'USD'
}
```

### Page-Specific Data

**Purchase Page URL Parameters**:
```
Purchase.html?product=Duo%20Pack&price=54&quantity=24
```

These parameters are parsed and used for e-commerce tracking.

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Page view tracking completeness
*For any* page load on the website, the Analytics System should record exactly one page view event with the correct page title and URL.
**Validates: Requirements 1.1**

### Property 2: Tracking code non-blocking
*For any* page with analytics tracking code, the page should render and become interactive even if the analytics script fails to load or is blocked.
**Validates: Requirements 1.4, 1.5**

### Property 3: Event data integrity
*For any* custom event tracked (CTA clicks, product selections), the event data should include all required parameters (event_category, event_label) and the data should match the user's actual interaction.
**Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5**

### Property 4: E-commerce funnel consistency
*For any* product selection that leads to checkout, the product details (name, price, quantity) should remain consistent across all funnel events (select_item → begin_checkout → view_cart).
**Validates: Requirements 3.1, 3.2, 3.3, 3.5**

### Property 5: URL parameter parsing accuracy
*For any* Purchase page load with URL parameters, the parsed product data should exactly match the values passed in the query string.
**Validates: Requirements 3.3**

### Property 6: Session continuity
*For any* user navigating between multiple pages, the Analytics System should maintain the same session ID and user ID across all page views within the session.
**Validates: Requirements 1.3**

### Property 7: Privacy compliance
*For any* analytics data collection, the system should not capture personally identifiable information (PII) such as email addresses, names, or phone numbers without explicit consent.
**Validates: Requirements 5.4**

### Property 8: Tracking code consistency
*For any* two pages on the website, the base analytics tracking code structure should be identical except for page-specific custom events.
**Validates: Requirements 6.1**

## Error Handling

### Tracking Script Load Failures

**Scenario**: GA4 script fails to load (network error, ad blocker, etc.)

**Handling**:
```javascript
// Graceful degradation - check if gtag exists before calling
function safeTrackEvent(eventName, params) {
  if (typeof gtag === 'function') {
    gtag('event', eventName, params);
  } else {
    console.warn('Analytics not available:', eventName, params);
  }
}
```

### Invalid E-commerce Data

**Scenario**: Product data is missing or malformed

**Handling**:
```javascript
function trackProductSelection(productName, price, quantity) {
  // Validate data before tracking
  if (!productName || !price || !quantity) {
    console.error('Invalid product data:', { productName, price, quantity });
    return;
  }
  
  gtag('event', 'select_item', {
    'items': [{
      'item_name': productName,
      'price': parseFloat(price),
      'quantity': parseInt(quantity),
      'currency': 'USD'
    }]
  });
}
```

### URL Parameter Parsing Errors

**Scenario**: Purchase page loaded without required parameters

**Handling**:
```javascript
function parseProductFromURL() {
  const params = new URLSearchParams(window.location.search);
  const product = params.get('product');
  const price = params.get('price');
  const quantity = params.get('quantity');
  
  if (!product || !price || !quantity) {
    console.warn('Missing product parameters in URL');
    return null;
  }
  
  return { product, price: parseFloat(price), quantity: parseInt(quantity) };
}
```

## Testing Strategy

### Manual Testing

**Page View Tracking**:
1. Load each page and verify page view appears in GA4 Real-Time report
2. Navigate between pages and verify session continuity
3. Test with ad blocker enabled to verify graceful degradation

**CTA Button Tracking**:
1. Click each "Shop Now", "Get Vero", and "Learn More" button
2. Verify events appear in GA4 Real-Time Events report
3. Confirm event parameters are correct (location, destination)

**E-commerce Funnel**:
1. Select a product pack and verify "select_item" event
2. Click "Select Pack" and verify "begin_checkout" event
3. Land on Purchase page and verify "view_cart" event
4. Complete checkout and verify "purchase" event
5. Confirm all events have consistent product data

**Content Engagement**:
1. Navigate product carousel and verify tracking
2. Scroll through page sections and verify scroll events
3. Expand FAQ/nutrition details and verify interaction events

### Browser Testing

Test across:
- Chrome (desktop & mobile)
- Safari (desktop & mobile)
- Firefox
- Edge

Verify:
- Tracking code loads without errors
- Events fire correctly
- Page performance is not degraded

### Performance Testing

**Metrics to Monitor**:
- Page load time (should not increase > 200ms)
- Time to Interactive (TTI)
- First Contentful Paint (FCP)

**Tools**:
- Chrome DevTools Performance tab
- Google PageSpeed Insights
- WebPageTest

### Privacy Testing

**Verify**:
- IP anonymization is enabled
- No PII is captured in events
- Tracking respects Do Not Track settings (if implemented)
- GDPR compliance (GA4 default settings)

### Integration Testing

**GA4 Dashboard Verification**:
1. Verify all custom events appear in Events report
2. Confirm e-commerce events populate E-commerce reports
3. Check that conversion funnels can be created
4. Verify traffic source attribution works correctly

## Implementation Notes

### Pages Requiring Analytics

All pages need base tracking code:
- index.html
- Products.html
- Science.html
- Nutrition.html
- Our-Story.html
- Purchase.html
- checkout.html
- Contact.html

### Measurement ID Placeholder

Use `G-XXXXXXXXXX` as placeholder in code. The actual Measurement ID will be provided after GA4 property setup.

### Event Naming Conventions

- Use lowercase with underscores: `cta_click`, `select_item`
- Follow GA4 recommended event names where applicable
- Keep custom event names descriptive and consistent

### Performance Considerations

- All tracking scripts load asynchronously
- Event tracking functions are lightweight
- No tracking calls block user interactions
- Failed tracking calls don't throw errors to user

### Future Enhancements

Potential additions not in current scope:
- Google Tag Manager for easier tag management
- Custom dimensions for user segments
- Enhanced scroll tracking with percentage thresholds
- Form field interaction tracking
- Exit intent tracking
- A/B testing integration
