1. student logs in
2. clicks on IGNOUMate+
3. goes on /pricing
4. one pricing plan
5. clicks on buy now
6. checks for if user is already subscribed, return to /dashboard if already subscribed w/ toast
7. asks to provide a mobile no. and referral id
8. sends api request to initiate payment w/ response of 201 and paymentRedirectUrl
9. redirect user to redirectUrl
10. student pays
11. webhook `/api/transactions/callback` is hit by payment_gateway service (callbackUrl)
12. update the transaction details to status=completed
13. add transaction to student's transaction list
14. redirect to `/payments/:id` (redirectUrl)
15. fetch the status and update the cookie
16. done
