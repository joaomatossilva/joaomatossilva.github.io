---
published: true
title: "Why I love micro-Orms"
layout: post
---

This is the main reason for me to not use Ado.Net directly anymore. With micro-Orms I can write this legacy code:

        public IEnumerable<User> GetUsersWithInvalidAttempts(int minAttempts)
        {
            using (var conn = CreateConnection())
            {
                var query = "select Id, Name, LastLogin, Active from users where InvalidLoginAttempts >= @minAttempts";
                var command = new SqlCommand(query, conn) {CommandType = CommandType.Text};
                command.Parameters.Add(new SqlParameter("minAttempts", minAttempts));

                using (var reader = command.ExecuteReader())
                {
                    while (reader.Read())
                    {
                        yield return new User
                                     {
                                         Id = reader.GetInt32(reader.GetOrdinal("Id")),
                                         Name = reader.GetString(reader.GetOrdinal("Name")),
                                         LastLogin = reader.IsDBNull(reader.GetOrdinal("LastLogin")) ? 
                                            (DateTime?)null : 
                                            reader.GetDateTime(reader.GetOrdinal("LastLogin")),
                                         Active = reader.GetBoolean(reader.GetOrdinal("Active"))
                                     };
                    }
                }
            }
        }


Into this:

        public IEnumerable<User> GetUsersWithInvalidAttempts(int minAttempts)
        {
            using (var conn = CreateConnection())
            {
                var query = "select Id, Name, LastLogin, Active from users where InvalidLoginAttempts >= @minAttempts";
                return conn.Query<User>(query, new {minAttempts}, commandType: CommandType.Text);
            }
        }
        
It just works.... 
In this case I was using [Dapper.Net](https://github.com/StackExchange/dapper-dot-net) but others are pretty similar.

Just for clarity, this is the class User:

    public class User
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime? LastLogin { get; set; }
        public bool Active { get; set; }
    }