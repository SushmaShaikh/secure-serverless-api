# Manual test log

Manual verification performed against a deployed `secure-serverless-api` CloudFormation
stack, using the AWS CLI, Postman, and PowerShell.

## GET /hello (public)

- Called with no Authorization header.
- **Result:** `200 OK`, body `Hello from secure-serverless-api`. Confirms the route is
  intentionally public and the Lambda + API Gateway wiring works end to end.

## POST /items (Cognito-protected)

- Created a test user via `aws cognito-idp admin-create-user` /
  `admin-set-user-password`, obtained an `IdToken` via
  `aws cognito-idp initiate-auth` (`USER_PASSWORD_AUTH` flow).
- Called with a valid Bearer `IdToken` and JSON body `{"id": "1", "name": "First Item"}`.
- **Result:** `201 Created`, item written to DynamoDB.

## GET /items/{id} (Cognito-protected)

- Called `GET /items/1` with a valid Bearer `IdToken`.
- **Result:** `200 OK` with the item created above.
- Called the same route with **no** Authorization header.
- **Result:** `401 Unauthorized` - confirms the JWT authorizer actually rejects
  unauthenticated requests rather than passing them through.

## API throttling

- Stage-level `ThrottlingRateLimit`/`ThrottlingBurstLimit` set on the `$default` stage.
- Attempted to trigger a `429` via rapid-fire requests (Postman Collection Runner,
  PowerShell background jobs) - not reproduced locally, likely because none of those
  tools fire requests fast/concurrently enough from a single machine to exceed even a
  temporarily-lowered limit (rate=2, burst=2).
- **Result:** not visually confirmed end-to-end; the setting itself is standard,
  well-documented API Gateway behavior. Treated as configured-but-not-demonstrated.
  Revisit with a proper load-testing tool (e.g. `hey`, `k6`, Artillery) if a live
  demonstration is needed later.

## CloudWatch alarms

- `LambdaErrorsAlarm` (watches `AWS/Lambda Errors` for the Hello function) and
  `ApiGateway5xxAlarm` (watches `AWS/ApiGateway 5xxError` for this API/stage), both
  publishing to an `AlarmTopic` SNS topic on trigger.
- Subscribed a personal email to `AlarmTopic` via `aws sns subscribe`, confirmed via
  the link in AWS's confirmation email. Verified with `aws sns list-subscriptions-by-topic`
  that the subscription is active (no longer `PendingConfirmation`).
- **Result:** subscription confirmed and active; did not force a real Lambda error or
  5xx to see an actual alarm-triggered email arrive (would take several minutes for
  CloudWatch to evaluate and transition state). Treated as configured-and-subscribed,
  not live-fired - same honesty standard as the throttling entry above.
