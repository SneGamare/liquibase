package com.example;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.beans.factory.annotation.Autowired;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ResultSet;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@Testcontainers
public class LiquibaseIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("testuser")
            .withPassword("testpass");

    @DynamicPropertySource
    static void overrideProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.liquibase.enabled", () -> true);
    }

    @Autowired
    private DataSource dataSource;

    @Test
    void testLiquibaseRanAndTableExists() throws Exception {
        try (Connection conn = dataSource.getConnection();
             ResultSet rs = conn.getMetaData().getTables(null, null, "customer", null)) {
            assertThat(rs.next()).isTrue(); // customer table should exist
        }
    }
}




# src/test/resources/application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb;MODE=PostgreSQL
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver
spring.liquibase.change-log=classpath:/db/changelog/db.changelog-master.yaml
spring.jpa.hibernate.ddl-auto=none




@SpringBootTest
@ActiveProfiles("test")
public class LiquibaseH2Test {
    @Autowired
    private DataSource dataSource;

    @Test
    void tableExists() throws Exception {
        try (Connection conn = dataSource.getConnection();
             ResultSet rs = conn.getMetaData().getTables(null, null, "CUSTOMER", null)) {
            assertThat(rs.next()).isTrue();
        }
    }
}
 
