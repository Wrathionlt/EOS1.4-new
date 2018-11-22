# EOS1.4-new
最新最全的EOS开发教程
#介绍
Daniel Larimer 在他的博客介绍了EOS新的智能合约架构（EOS团队的开发速度实在是太吓人，根本追不上）。他给出了最简单的一个新币种的智能合约代码，仅有49行就能完成一个新币种的开发，一个新的“爱息欧”就诞生了，相比之前的官方“发币”代码，又简化了很多，让我们一步一步实现吧。。

首先实现私有成员，建立一个 account 结构体，这个结构体里保存的是所有持有我们这种代币的人的账户和余额。

private:
//account 结构体 
struct account {
EOS 账户名
account_name owner;
余额
uint64_t balance;
主键
uint64_t primary_key()const { return owner; }
下一步 我们要利用 Boost 库中的多索引列表，将上面声明的结构体放入一个列表中，方便查询和修改。

eosio::multi_index<N(accounts), account> _accounts;
接着，实现 add_balance() 函数，这个私有函数的目的是给特定的 EOS 账户增加特定的代币。

void add_balance( account_name payer, account_name to, uint64_t q ) {
     //在列表中查询，看要收币的用户是否已经在列表中。
     auto toitr = _accounts.find( to );
     //如果不在列表中，说明用户从未持有过这种币，要将用户加入列表
     if( toitr == _accounts.end() ) {
        增加一个用户
       _accounts.emplace( payer, [&]( auto& a ) {
          a.owner = to;
          因为之前没有这种币，用户名下的余额为要接收的数量
          a.balance = q;
       });
       //如果用户在列表中，说明已经持有或持有过这种币
     } else {
       _accounts.modify( toitr, 0, [&]( auto& a ) {
           //直接将余额增加要转入的数量
          a.balance += q;
          //判断用户余额是否溢出（余额增加了q，之后数量应该大于q）
          eosio_assert( a.balance >= q, "overflow detected" );
       });
     }
  }
之后就要实现公有方法了，首先是构造函数，别忘了初始化 _accounts 列表。

public:
simpletoken( account_name self )
:contract(self),_accounts( _self, _self){}
实现公有的 transfer（转账）函数，将代币从一个账户转移到另一个账户。

void transfer( account_name from, account_name to, uint64_t quantity ) {
     //从付款方获取权限
     require_auth( from );
     //从列表中搜索发币方账户
     const auto& fromacnt = _accounts.get( from );
     //验证付款方余额，是否透支
     eosio_assert( fromacnt.balance >= quantity, "overdrawn balance" );
     //从付款方减去代币
     _accounts.modify( fromacnt, from, [&]( auto& a ){ a.balance -= quantity; } );
     //收款方增加代币（之前实现的私有函数）
     add_balance( from, to, quantity );
  }
OK，是不是以为大功告成了？还有最重要的 issue（发行）函数，要不从哪“印钱？”

void issue( account_name to, uint64_t quantity ) {
     //得到合约主人的权限
     require_auth( _self );
     //直接印钱
     add_balance( _self, to, quantity );
最后一步，将我们的 transfer 和 issue 函数接口提供给 EOS 系统，通过一个宏就可以快速实现。

EOSIO_ABI( simpletoken, (transfer)(issue) )
这个宏是咋回事？我们看看 dispacher.hpp 文件中对这个宏的定义,其实是替开发者实现了 apply 函数，使得开发者可以专注于业务逻辑。

define EOSIO_ABI( TYPE, MEMBERS ) \
extern "C" { \
void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
auto self = receiver; \
if( code == self ) { \
TYPE thiscontract( self ); \
switch( action ) { \
EOSIO_API( TYPE, MEMBERS ) \
} \
eosio_exit(0); \
} \
} \
} \
大功告成，看看全部的代码吧，是不是49行就搞定了？不过 EOS 表示以后会有系统的标准代币，连以上的具体逻辑都不用我们实现了，不过这段代码对系统学习 EOS 智能合约架构还是很有意义的。“发币”代码，又简化了很多，让我们一起来看看吧。
Daniel Larimer 在他的博客介绍了EOS新的智能合约架构（EOS团队的开发速度实在是太吓人，根本追不上）。他给出了最简单的一个新币种的智能合约代码，仅有49行就能完成一个新币种的开发，一个新的“爱息欧”就诞生了让。我们一步一步实现吧。
首先实现私有成员，建立一个 account 结构体，这个结构体里保存的是所有持有我们这种代币的人的账户和余额。

 private:
      //account 结构体 
      struct account {
         EOS 账户名
         account_name owner;
         余额
         uint64_t     balance;
         主键
         uint64_t primary_key()const { return owner; }
下一步 我们要利用 Boost 库中的多索引列表，将上面声明的结构体放入一个列表中，方便查询和修改。

    eosio::multi_index<N(accounts), account> _accounts;
接着，实现 add_balance() 函数，这个私有函数的目的是给特定的 EOS 账户增加特定的代币。

    void add_balance( account_name payer, account_name to, uint64_t q ) {
         //在列表中查询，看要收币的用户是否已经在列表中。
         auto toitr = _accounts.find( to );
         //如果不在列表中，说明用户从未持有过这种币，要将用户加入列表
         if( toitr == _accounts.end() ) {
            增加一个用户
           _accounts.emplace( payer, [&]( auto& a ) {
              a.owner = to;
              因为之前没有这种币，用户名下的余额为要接收的数量
              a.balance = q;
           });
           //如果用户在列表中，说明已经持有或持有过这种币
         } else {
           _accounts.modify( toitr, 0, [&]( auto& a ) {
               //直接将余额增加要转入的数量
              a.balance += q;
              //判断用户余额是否溢出（余额增加了q，之后数量应该大于q）
              eosio_assert( a.balance >= q, "overflow detected" );
           });
         }
      }
之后就要实现公有方法了，首先是构造函数，别忘了初始化 _accounts 列表。

public:
      simpletoken( account_name self )
      :contract(self),_accounts( _self, _self){}
实现公有的 transfer（转账）函数，将代币从一个账户转移到另一个账户。

    void transfer( account_name from, account_name to, uint64_t quantity ) {
         //从付款方获取权限
         require_auth( from );
         //从列表中搜索发币方账户
         const auto& fromacnt = _accounts.get( from );
         //验证付款方余额，是否透支
         eosio_assert( fromacnt.balance >= quantity, "overdrawn balance" );
         //从付款方减去代币
         _accounts.modify( fromacnt, from, [&]( auto& a ){ a.balance -= quantity; } );
         //收款方增加代币（之前实现的私有函数）
         add_balance( from, to, quantity );
      }
OK，是不是以为大功告成了？还有最重要的 issue（发行）函数，要不从哪“印钱？”

    void issue( account_name to, uint64_t quantity ) {
         //得到合约主人的权限
         require_auth( _self );
         //直接印钱
         add_balance( _self, to, quantity );
最后一步，将我们的 transfer 和 issue 函数接口提供给 EOS 系统，通过一个宏就可以快速实现。

EOSIO_ABI( simpletoken, (transfer)(issue) )
这个宏是咋回事？我们看看 dispacher.hpp 文件中对这个宏的定义,其实是替开发者实现了 apply 函数，使得开发者可以专注于业务逻辑。

#define EOSIO_ABI( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      auto self = receiver; \
      if( code == self ) { \
         TYPE thiscontract( self ); \
         switch( action ) { \
            EOSIO_API( TYPE, MEMBERS ) \
         } \
         eosio_exit(0); \
      } \
   } \
} \
大功告成，看看全部的代码吧，是不是49行就搞定了？不过 EOS 表示以后会有系统的标准代币，连以上的具体逻辑都不用我们实现了，不过这段代码对系统学习 EOS 智能合约架构还是很有意义的。
