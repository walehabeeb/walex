# walex
/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/Classes/Class.java to edit this template
 */
package query;

import Person.Person;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.PreparedStatement;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 *
 * @author user
 */
public class PersonQuery {

    static final String DB_URL
            = "jdbc:mysql://localhost:3306/testdb"; //database connection address
    static final String DB_DRV
            = "com.mysql.jdbc.Driver"; //mysqljava database connectivity library
    static final String DB_USER = "wale"; //database username
    static final String DB_PASSWD = "password"; //database password

    private Connection connection;
    private PreparedStatement preparedStatement;
    private PreparedStatement insertNewPerson;
    private PreparedStatement selectAllPeople;
    private PreparedStatement selectPeopleByLastName;

    //constructor
    public PersonQuery() {
        try {
            connection
                    = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWD);
            //selects create query that selects all entries inthe addresbook 
            selectAllPeople
                    = connection.prepareStatement("select * from addresses order by lastName, firstName"
                    );
            //create insert that entries with last nmaes
            // that begins with the specified characters

            selectPeopleByLastName = connection.prepareStatement("select from addresses where lastName like ?"
                    + "order by lastName, firstName");

            //ctreate insert that adds a new entry into database 
            insertNewPerson = connection.prepareStatement("insert into addresses "
                    + "(firstName, lastName, email, phoneNumber)"
                    + "values (?, ?, ?, ?)");
        } catch (SQLException sQLException) {
            sQLException.printStackTrace();
            System.exit(1);
        }
    }

    //select all of the addresses in the database 
    public List<Person> getAllPeople() throws SQLException {
        //executeQuery returns resultSet containing matching entries 
        try (ResultSet resultSet = selectAllPeople.executeQuery()) {
            List<Person> resultList = new ArrayList<>();

            while (resultSet.next()) {
                resultList.add(new Person(
                        resultSet.getInt("addressID"),
                        resultSet.getString("firstName"),
                        resultSet.getString("lastName"),
                        resultSet.getString("email"),
                        resultSet.getString("phoneNumber")
                ));
            }
            return resultList;
        } catch (SQLException sQLException) {
            sQLException.printStackTrace();
        }
        return null;
    }

    //selct persobn by last name 
    public List<Person> getPeopleByLastName(String lastName) {
        try {
            selectPeopleByLastName.setString(1, lastName); // set lastName
        } catch (SQLException sQLException) {
            sQLException.printStackTrace();
            return null;
        }

        // executeQuery returns ResultSet containing matching entries 
        try (ResultSet resultSet = selectPeopleByLastName.executeQuery()) {
            List<Person> resultList = new ArrayList<>();
            while (resultSet.next()) {

                resultList.add(new Person(
                        resultSet.getInt("addressID"),
                        resultSet.getString("firstName"),
                        resultSet.getString("lastName"),
                        resultSet.getString("email"),
                        resultSet.getString("phoneNumber")));
            }
            return resultList;

        } catch (SQLException ex) {
            Logger.getLogger(PersonQuery.class.getName()).log(Level.SEVERE, null, ex);

            return null;
        }

    }

    // add an entry 
    public int addPerson(int addressID, String firstName,
            String lastName, String email, String phoneNumber) {

        // insert the new entry; return # of rowa updated 
        try {
            // set paremeters
            insertNewPerson.setString(1, firstName);
            insertNewPerson.setString(2, lastName);
            insertNewPerson.setString(3, email);
            insertNewPerson.setString(4, phoneNumber);

            return insertNewPerson.executeUpdate();
        } catch (SQLException ex) {
            Logger.getLogger(PersonQuery.class.getName()).log(Level.SEVERE, null, ex);
            return 0;
        }

    }
    
    public void close(){
        try{
            connection.close();
        }
        catch(SQLException sQLException){
            sQLException.printStackTrace();
        }
    }
}
