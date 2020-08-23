# Columnar File Format

[we-taking-only-half-advantage-columnar-file-format-eric-sun](https://www.linkedin.com/pulse/we-taking-only-half-advantage-columnar-file-format-eric-sun/)

- Even though sorting is an expensive operation, it's worth it if you are using columnar file format. 
- Sorting reduces scan on big data
- Couple stratigies for sorting. Usually sort via column with many characters or important column like `user_id`.
- Apparantly Spark 2.+ had vectorization reader and optimization support. Had no idea 🤷‍♂️
- Split size and JVM container memory is trickier with compressed data.