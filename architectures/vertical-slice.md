# Victor Rentea

https://github1s.com/victorrentea/clean-architecture/blob/HEAD/clean-application/src/main/java/victor/training/clean/verticalslice/SearchCustomerUseCase.java

```java

@SuppressWarnings("JpaQlInspection")
@RequiredArgsConstructor
//@RestController
public class SearchCustomerUseCase {
    private final EntityManager entityManager;

    @Value
    // in/out structures kept private, inaccessible from other Use-Cases!
    public static class SearchCustomerRequest { // JSON
        String name;
        String email;
        Long countryId;
    }

    @Value
    public static class SearchCustomerResponse { // JSON
        long id;
        String name;
        // TODO if we add 'email' to results => only this file is impacted
    }

    @Operation(description = "Customer Search")
    @PostMapping("customer/search")
    public List<SearchCustomerResponse> search(@RequestBody SearchCustomerRequest criteria) {
        String jpql = "SELECT new victor.training.clean.verticalslice.SearchCustomerUseCase$SearchCustomerResponse(c.id, c.name)" +
                " FROM Customer c " +
                " WHERE ";
        List<String> jpqlParts = new ArrayList<>();
        jpqlParts.add("1=1"); // alternatives: Criteria API ± Spring Specifications or Query DSL
        Map<String, Object> params = new HashMap<>();

        if (criteria.name != null) {
            jpqlParts.add("UPPER(c.name) LIKE UPPER('%' || :name || '%')");
            params.put("name", criteria.name);
        }

        if (criteria.email != null) {
            jpqlParts.add("UPPER(c.email) = UPPER(:email)");
            params.put("email", criteria.email);
        }

        if (criteria.countryId != null) {
            jpqlParts.add("c.country.id = :countryId");
            params.put("countryId", criteria.countryId);
        }

        String whereCriteria = join(" AND ", jpqlParts);
        var query = entityManager.createQuery(jpql + whereCriteria, SearchCustomerResponse.class);
        for (String paramName : params.keySet()) {
            query.setParameter(paramName, params.get(paramName));
        }
        return query.getResultList();
    }
}
```