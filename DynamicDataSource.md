###  关于springboot多数据源的切换问题

+ 主要核心思想是通过继承AbstractRoutingDataSource，实现其抽象方法determineCurrentLookupKey()，在获取新connection的时候会调用这个方法来获取本次操作的具体数据源对应的key
+ 为了保证线程安全，每个线程的key都放在其对应的ThreadLocal里，每次获取数据库连接connection的时候取出

#### 以上都是常规的东西，网上一大堆，那么本次遇到的问题其实很抽象
+ 通过testNG测试方法的时候，总会出现无法切换，通过debug以后发现，ThreadLocal的设置是正常的，也就是key是正确的，但是数据库连接切换却不成功。
+ 那么问题就在determineCurrentLookupKey()这个方法的触发上
+ 本次是使用spring namedJdbcTemplate来进行数据库操作，不断的进入下层方法，找到获取数据库connection的代码JdbcTemplate的execute方法中
```java
org.springframework.jdbc.core.JdbcTemplate#execute(org.springframework.jdbc.core.PreparedStatementCreator, org.springframework.jdbc.core.PreparedStatementCallback<T>)
```
具体的代码如下
```java
		Assert.notNull(psc, "PreparedStatementCreator must not be null");
		Assert.notNull(action, "Callback object must not be null");
		if (logger.isDebugEnabled()) {
			String sql = getSql(psc);
			logger.debug("Executing prepared SQL statement" + (sql != null ? " [" + sql + "]" : ""));
		}

		Connection con = DataSourceUtils.getConnection(getDataSource());
		PreparedStatement ps = null;
		try {
			Connection conToUse = con;
			if (this.nativeJdbcExtractor != null &&
					this.nativeJdbcExtractor.isNativeConnectionNecessaryForNativePreparedStatements()) {
				conToUse = this.nativeJdbcExtractor.getNativeConnection(con);
			}
			ps = psc.createPreparedStatement(conToUse);
			applyStatementSettings(ps);
			PreparedStatement psToUse = ps;
			if (this.nativeJdbcExtractor != null) {
				psToUse = this.nativeJdbcExtractor.getNativePreparedStatement(ps);
			}
			T result = action.doInPreparedStatement(psToUse);
			handleWarnings(ps);
			return result;
		}
		catch (SQLException ex) {
			// Release Connection early, to avoid potential connection pool deadlock
			// in the case when the exception translator hasn't been initialized yet.
			if (psc instanceof ParameterDisposer) {
				((ParameterDisposer) psc).cleanupParameters();
			}
			String sql = getSql(psc);
			psc = null;
			JdbcUtils.closeStatement(ps);
			ps = null;
			DataSourceUtils.releaseConnection(con, getDataSource());
			con = null;
			throw getExceptionTranslator().translate("PreparedStatementCallback", sql, ex);
		}
		finally {
			if (psc instanceof ParameterDisposer) {
				((ParameterDisposer) psc).cleanupParameters();
			}
			JdbcUtils.closeStatement(ps);
			DataSourceUtils.releaseConnection(con, getDataSource());
		}
```

关键的获取连接代码是这一条
```java
Connection con = DataSourceUtils.getConnection(getDataSource());
```
那么再往下进入方法以后其实主要的实现是在doGetConnection方法中，代码如下：
```java
/**
	 * Actually obtain a JDBC Connection from the given DataSource.
	 * Same as {@link #getConnection}, but throwing the original SQLException.
	 * <p>Is aware of a corresponding Connection bound to the current thread, for example
	 * when using {@link DataSourceTransactionManager}. Will bind a Connection to the thread
	 * if transaction synchronization is active (e.g. if in a JTA transaction).
	 * <p>Directly accessed by {@link TransactionAwareDataSourceProxy}.
	 * @param dataSource the DataSource to obtain Connections from
	 * @return a JDBC Connection from the given DataSource
	 * @throws SQLException if thrown by JDBC methods
	 * @see #doReleaseConnection
	 */
	public static Connection doGetConnection(DataSource dataSource) throws SQLException {
		Assert.notNull(dataSource, "No DataSource specified");

		ConnectionHolder conHolder = (ConnectionHolder) TransactionSynchronizationManager.getResource(dataSource);
		if (conHolder != null && (conHolder.hasConnection() || conHolder.isSynchronizedWithTransaction())) {
			conHolder.requested();
			if (!conHolder.hasConnection()) {
				logger.debug("Fetching resumed JDBC Connection from DataSource");
				conHolder.setConnection(dataSource.getConnection());
			}
			return conHolder.getConnection();
		}
		// Else we either got no holder or an empty thread-bound holder here.

		logger.debug("Fetching JDBC Connection from DataSource");
		Connection con = dataSource.getConnection();

		if (TransactionSynchronizationManager.isSynchronizationActive()) {
			logger.debug("Registering transaction synchronization for JDBC Connection");
			// Use same Connection for further JDBC actions within the transaction.
			// Thread-bound object will get removed by synchronization at transaction completion.
			ConnectionHolder holderToUse = conHolder;
			if (holderToUse == null) {
				holderToUse = new ConnectionHolder(con);
			}
			else {
				holderToUse.setConnection(con);
			}
			holderToUse.requested();
			TransactionSynchronizationManager.registerSynchronization(
					new ConnectionSynchronization(holderToUse, dataSource));
			holderToUse.setSynchronizedWithTransaction(true);
			if (holderToUse != conHolder) {
				TransactionSynchronizationManager.bindResource(dataSource, holderToUse);
			}
		}

		return con;
	}
```
到了这一步，基本就柳暗花明了，问题就在于第一个条件判断，主要作用是判断当前的dataSource是否已经有获取过connection了且这个connection是否正处于事务下，结果问题原因就出在这个事务判断上，testNG的方法在执行前就会开始一个事务，而这个时候注解还未生效，ThreadLocal中的值为默认数据源。

#### 那么能不能把注解加在testNG方法上设置ThreadLocal中的key呢
+ 答案是不行，不知道为啥，看来这个开启事务的判定要先于aop拦截的执行顺序，有空再说吧，这部分感觉再研究也没啥意义，反正正常执行的时候也不会用testNG来搞，真的傻卵这个东西，浪费了这么久时间。
+ 困的一比，国足还0-1落后了，呜呜呜，希望明天会更好QAQ 

##### 2021/9/7 23:54 平平无奇的一天

