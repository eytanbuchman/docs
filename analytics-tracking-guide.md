# Analytics Event Tracking Guide

## Rule: Every Significant Feature Requires Analytics Events

**When implementing a new feature or significant enhancement, you MUST include appropriate analytics events.**

## Event Implementation Process

### 1. Pre-Implementation Planning

Before starting implementation, identify which user actions or system events should be tracked:

- **User-initiated actions**: Form submissions, button clicks, selections
- **System events**: Process completions, errors, threshold crossings
- **Page/view loads**: Entry points and key page navigations
- **Success/failure outcomes**: Task completions, error states

### 2. During Implementation

As you build the feature:

1. Use the `trackX` functions from `src/lib/posthog-events.ts`
2. For new event types, add a new tracking function to that file
3. Include relevant properties with each event:
   - User context (if applicable)
   - Feature-specific details
   - Outcome information (success/failure, etc.)

```typescript
// Example event structure
trackNewFeatureAction(actionType, {
  feature_area: "area_name",
  action_source: "source_description",
  outcome: "success|failure",
  additional_context: value
});
```

### 3. Pre-Check-in Checklist

Before submitting your work, complete this checklist:

- [ ] Identified all key user interactions for tracking
- [ ] Added appropriate tracking calls at each interaction point
- [ ] Included relevant context with each tracked event
- [ ] Tested that events appear correctly in PostHog
- [ ] Documented any new tracking functions added

### 4. Code Review Requirements

Pull requests that add new features must include:

1. The completed analytics checklist above
2. A list of new events added and their purpose
3. Confirmation that events appear correctly in PostHog

Reviewers should verify that appropriate analytics have been implemented.

## Common Events to Track

### Core User Journey
- Page loads (`trackPageLoad`)
- User signup/login (`trackSignup`, `trackLogin`)
- Feature usage start/completion
- Error states

### Feature-Specific Events
- Asset-related: Uploads, downloads, sharing
- Workflow completion steps
- Configuration changes
- Key decision points

## Using Existing Tracking Functions

The following tracking functions are available in `src/lib/posthog-events.ts`:

- `identifyUser()` - Sets user identity with email and workspace
- `trackPageLoad(pageName, props)` - Tracks page views with context
- `trackSignup(email, workspaceName)` - Tracks user signup
- `trackLogin(email)` - Tracks user login
- `trackNewAssistantCreated(name, type)` - Tracks assistant creation
- `trackAssetUpload(assetId, fileName, fileType, fileSize)` - Tracks file uploads

## Adding New Tracking Functions

When adding a new event type:

1. Follow the existing pattern in `posthog-events.ts`
2. Use consistent naming: `track{EventName}`
3. Include appropriate error handling
4. Log success/failure to console
5. Document the function with JSDoc comments

```typescript
/**
 * Track [description of event]
 */
export function trackNewFeature(requiredParam: string, optionalProps: Record<string, any> = {}) {
  try {
    posthog.capture('new_feature_event', {
      primary_property: requiredParam,
      ...optionalProps
    });
    console.log(`[PostHog] Tracked new feature: ${requiredParam}`);
  } catch (error) {
    console.error('[PostHog] Error tracking new feature:', error);
  }
}
```

## Analytics Review Process

During feature development and review:

1. **Planning Phase**: Identify key events to track
2. **Implementation**: Add appropriate tracking
3. **Review**: Verify events are tracked properly
4. **Post-Launch**: Confirm data is flowing to PostHog

## Questions to Ask When Adding Events

- What user action or system event is occurring?
- What context is important to capture?
- How will this data be used for analysis?
- Does this event provide insights on feature adoption or usage patterns?
- Is there any user feedback or outcome to record? 