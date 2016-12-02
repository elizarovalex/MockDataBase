MockDataBase
=======
A simple framework for mocking a database in .Net

## Example 1

### Main code:

	public interface IDbAccessor
	{
		IDataReader Select();
	}

	public class Entity
	{
		public int Field1 { get; set; }
	}

	public class Repository
	{
		public int DoWork(IDbAccessor dbAccessor)
		{
			var obj = MapToObject<Entity>(dbAccessor.Select());
			return obj.Field1 == 0 ? 3 : 5;
		}
	}

### Test for class Repository

	[Test]
	public void Test()
	{
		// Given
		var expecedEntity = new Entity { Field1 = 10 };

		var moq = new Moq<IDbAccessor>();
		moq.Setup(a => Select()).Return(expecedEntity.ToDataReader());

		// When
		var target = new Repository();
		var value = target.DoWork(moq.Object);

		// Then
		Assert.AreEqual(5, value);
	}

## Example 2

### Main code:

    public interface IDbFactory
	{
		IDbConnection CreateConnection();
	}

	public class Repository
	{
		public int DoWork(IDbFactory factory)
		{
			var value1 = 0;
			int value2;
			using (var conn = factory.CreateConnection())
			{
				conn.Open();

				using (var cmd = conn.CreateCommand())
				{
					cmd.CommandText = "select 1 as Field1 union select 2 as Field1";
					using (var reader = cmd.ExecuteReader())
					{
						while (reader.Read())
						{
							value1 += reader.GetInt32(reader.GetOrdinal("Field1"));
						}
					}
				}

				using (var cmd = conn.CreateCommand())
				{
					cmd.CommandText = "select 3 as Field2";
					using (var reader = cmd.ExecuteReader())
					{
						reader.Read();
						value2 = reader.GetInt32(reader.GetOrdinal("Field2"));
					}
				}
			}

			return value1 * value2;
		}
	}

### Test for class Repository

		[Test]
		public void Test()
		{
			// Given
			var db = new MockDb()
				.NewReader("Field1")
					.NewRow(1)
					.NewRow(2)
				.NewReader("Field2")
					.NewRow(3);

			var moq = new Moq<IDbFactory>();
			moq.Setup(a => CreateConnection()).Return(db.AsDbConnection());

			// When
			var target = new Repository();
			var value = target.DoWork(moq.Object);

			// Then
			Assert.AreEqual(9, value);
		}

## License
MockDataBase is licensed under [MIT License](http://en.wikipedia.org/wiki/MIT_License)
