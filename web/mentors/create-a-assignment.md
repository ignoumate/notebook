# Authorized on apk

1. visits /dashboard/my-assignments
2. clicks on add new
3. add course code, TEE, description (<= 50 words)
4. add pdf (< 5 mb)
5. clicks upload
6. post request to POST /api/mentors/assignments
7. creates a new Assignment entry in db
8. return 201 response
9. done
