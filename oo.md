# 设计模式 (基于typescript)

* ### 工厂

    基于抽象工厂模式的filter设计
        
    ```typescript

    interface Midware {
      (ctx,next) : Promise<any>
    }

    interface FilterAble {
      filter(type:string) : Midware
    }

    abstract class Filter implements FilterAble{
      abstract filter () : Midware
    }

    abstract class FilterFactory{
      abstract create(): Filter;
    }

    class CreateFilter extend Filter{
      filter(){
        return async (ctx,next)=>{
          let body = ctx.request.body;
          if(!body.username || !body.password){
            throw new Error('data require')
          }
          await next();
        }
      }
    }

    class UserFilterFactory extend FilterFactory{
      create(type:string) : Filter {
        let filter;
        switch (type) {
          case 'create':
            filter = new CreateFilter();
            break;
          default : 
            filter = new CreateFilter();
            break;
        }
        return filter;
      }
    }

    let filter = new UserFilterFactory().create('create');
    user.post('/',filter.filter(),user.create);
    ```
