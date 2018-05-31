# Prototyping


## Implementation 

All implementations of [gdb-use-case](https://github.com/Ciberth/gdb-use-case) are re-used. Only the extra steps will be discussed. In other words the new generic database dba charm (gdb-dba charm from now on) and the generic database dba interface layer (gdb-dba interface layer from now). For a prototype of the implementation refer to [gdb-dba charm]().

### Metadata.yaml of gdba-dba charm

```yaml
name: gdb-dba
# ...
provides:
  website:
    interface: http
  dba:
    interface: gdb-dba
requires:
  database:
    interface: generic-database
```

### Metadata.yaml of webapp charm

```yaml
name: gdb-dba
# ...
provides:
  website:
    interface: http
requires:
  database:
    interface: generic-database
  dba:
    interface: gdb-dba
```

### Code in webapp charm to populate the database

```python

# ...

@when('endpoint.dba.available')
def populate_database():
    dba = endpoint_from_flag('endpoint.dba.available')
    # could be read from a file
    query = """
        CREATE TABLE Persons (id int, lastname varchar(255), firstname varchar(255));
        INSERT INTO Persons (id, lastname, firstname) VALUES (0, "admin", "admin");
        INSERT INTO Persons (id, lastname, firstname) VALUES (1, "test", "test");
    """
    # Example to run a query
    dba.execute_query(query)

    # Example to add a user
    """
    parameters: 1. user ; 
                2. password ; 
                3. hostname of database (already known thanks to generic database)
                4. databasename (already known thanks to generic database)
                4. privileges
    """
    dba.add_user('user', 'password', database_info['hostname'], database_info['dbname'], "ALL PRIVILEGES")
    

@when('endpoint.dba.operation')
def check_on_errors():
    dba = endpoint_from_flag('endpoint.dba.operation')
    if dba.error():
        status_set('maintenance','Error occured: %s' % dba.error())
    else:
        status_set('active', 'Webapp ready!')
```

### gdb-dba interface layer requires side

```python
class GenericDatabaseDbaClient(Endpoint):
    
    @when('endpoint.{endpoint_name}.changed')
    def _handle_dba_is_available(self):
        set_flag(self.expand_name('endpoint.{endpoint_name}.available'))
    
    def execute_query(self, query):
        for relation in self.relations:
            relation.to_publish['query'] = query
    
    def add_user(self, user, password, hostname, dbname, privileges):
        for relation in self.relations:
            relation.to_publish['user'] = user
            relation.to_publish['password'] = password
            relation.to_publish['hostname'] = hostname
            relation.to_publish['dbname'] = dbname
            relation.to_publish['privileges'] = privileges
            
```

