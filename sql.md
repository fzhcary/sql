This is a very interesting sql puzzle
https://mystery.knightlab.com/walkthrough.html

solution
```
1. find out two witness person using two clues,

by street name and number

SELECT * FROM person
WHERE address_street_name = "Northwestern Dr"
ORDER BY address_number
DESC
LIMIT 3

by street name and name

SELECT * FROM person
WHERE address_street_name='Franklin Ave'
and name LIKE "%Annabel%"
LIMIT 3

2. find their interview

SELECT * FROM interview 
WHERE person_id in (16371, 14887)

3. find the criminal

SELECT * FROM get_fit_now_member m
JOIN person p
ON p.id=m.person_id
JOIN drivers_license l
ON p.license_id=l.id
WHERE m.id LIKE "48Z%" and
l.plate_number LIKE "%H42W%"

verify the check in time

SELECT * FROM get_fit_now_check_in
WHERE membership_id='48Z55'

```
