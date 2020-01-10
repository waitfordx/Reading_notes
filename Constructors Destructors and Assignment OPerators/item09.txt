绝不在构造和析构过程中调用 virtual 函数

01	不该在构造函数和析构函数期间调用 virtual 函数
	eg: 假设一个 class 集成体系，用来模拟股市交易，日志函数为 pure virtual 函数
	class Transaction{
	public:
		Transaction();
		virtual void logTransaction() const = 0;			// 做出一份因类型不同而不同的日志记录。
		
		...
	};
	Transaction::Transaction()
	{
		...
		logTransaction();									// 构造函数中调用了日志函数(virtual)
	}
	
	class BuyTransaction()
	{
	public:
		virtual void logTransaction() const;				// 日志函数
		...
	};
	
	BuyTransaction b;
	当执行上述语句时, BuyTransaction 的构造函数会被调用，但首先 base class 的构造函数会被先调用，Trasaction 会电泳 virtual 函数 logTransaction，这时候调用的版本是 Transaction 而不是 BuyTransaction 内的版本。（在 base class 构造期间， virtual 函数不是 virtual 函数）
	在 derived class 对象的 base class 构造期间，对象的类型是 base class 而不是 derived class。
	
02	侦测“构造函数或析构函数运行期间是否调用 virtual 函数”并不容易
	class Transaction{
	public：
		Transaction()
		{
			init();										// 调用 non-virtual 函数
		}
		virtual void logTransaction() const = 0;
		...
	private:
		void init()
		{
			...
			logTransaction();							// 这里调用 virtual
		}
	};
	这段代码可以通过编译器和连接器的检查。但是会导致错误的结果。
	确保构造函数和析构函数没有调用 virtual 函数，且它们调用的所有函数也符合这一约束。
	
03	调用 non-virtual 函数并让 derived class 向 base class 传递信息
	class Transaction{
	public:
		explicit Transaction(const std::string& logInfo);
		void logTransaction(const std::string& logInfo) const ;		// non-virtual 函数
		...
	};
	Transaction::Transaction(const std::string& logInfo)
	{
		...
		logTransaction(logInfo);									// 如今是个 non-virtual 调用
	}
	
	class BuyTransaction: public Transaction{
	public:
		BuyTransaction( parameters ) : Transaction(creatLogString ( parameters ) ) { }
		...
	private:
		static std::string creatLogString( parameters );
	};
	// 类外需要定义 static 成员 或 static 成员函数（static 会在其他对象之前定义）
	
04	总结
	在构造和析构期间不要调用 virtual 函数，因为这类调用不会下降到 derived classes