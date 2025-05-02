# Authorized

1. visits /assignments
2. filters based on programme and tee (july/dec)
3. show all available assignments
4. clicks on required assignment
5. opens up assignment product page /assignments/:id
6. clicks on buy now
7. order details page with total etc
8. clicks on proceed
9. fetch pgRedirectUrl from POST /api/assignments/buy
10. creates new transaction in db and return redirectUrl
11. redirect to payment_service url
12. student pays
13. webhook `/api/transactions/callback` is hit by payment_gateway service (callbackUrl)
14. update the transaction details to status=completed
15. add transaction to student's transaction list
16. redirect to `/payments/:id` (redirectUrl)
17. fetch the status and detail of the assignment
18. take pdf from s3 bucket and download it as pdf to student
19. done
