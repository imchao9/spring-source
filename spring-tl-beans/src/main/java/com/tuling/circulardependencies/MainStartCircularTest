package com.tuling.circulardependencies;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.support.RootBeanDefinition;

import java.lang.reflect.Field;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class MainStartAnnotated {
	// 1、定义一个BeanDefinitionMap
	private static Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();

	// 读取Bean定义，创建两个Bean并放到BeanDefinitionMap里
	private static void loadBeanDefinitions() {
		RootBeanDefinition rootBeanDefinition1 = new RootBeanDefinition(InstanceA.class);
		RootBeanDefinition rootBeanDefinition2 = new RootBeanDefinition(InstanceB.class);
		beanDefinitionMap.put("instanceA", rootBeanDefinition1);
		beanDefinitionMap.put("instanceB", rootBeanDefinition2);
	}


	public static void main(String[] args) throws InstantiationException, IllegalAccessException {
		// 加载BeanDefinition
		loadBeanDefinitions();

		// 注册Bean后置处理器


		// 构建Bean
		for (String key:beanDefinitionMap.keySet()) {
			getBean(key);
		}

		InstanceA instanceA = (InstanceA) getBean("instanceA");
		instanceA.say();

	}

	private static Map<String, Object> singletonObjects = new ConcurrentHashMap<>();
	// 二级缓存：为了将成熟Bean和纯净Bean分离，避免读取不完整的Bean
	private static Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>();

	private static Map<String, ObjectFactory> singletonFactories = new ConcurrentHashMap<>();

	//循环依赖标识
	private static Set<String> singletonsCurrentlyInCreation = new HashSet<>();


	public static Object getBean(String beanName) throws InstantiationException, IllegalAccessException {

		Object singletonBean = getSingletonBean(beanName);

		if (singletonBean != null) {
			return singletonBean;
		}

		if (singletonsCurrentlyInCreation.contains(beanName)) {
			singletonsCurrentlyInCreation.add(beanName);
		}

		//实例化
		RootBeanDefinition rootBeanDefinition = (RootBeanDefinition) beanDefinitionMap.get(beanName);
		Class<?> beanClass = rootBeanDefinition.getBeanClass();

		Object instanceBean = beanClass.newInstance();

		/**
		 * 再从二级缓存拿
		 * 有的话就说明是循环依赖
		 * 创建动态代理 Spring希望正常的Bean在初始化后创建
		 * 只在循环依赖的情况下在实例化后创建动态代理，所以需要判断当前是不是循环依赖
		 */
		singletonFactories.put(beanName, () -> new JdkProxyBeanPostProcessor().getEarlyBeanReference(earlySingletonObjects.get(beanName), beanName));

//		// 放到二级缓存中
//		earlySingletonObjects.put(beanName, instanceBean);

		//属性赋值
		Field[] fields = instanceBean.getClass().getDeclaredFields();

		for (Field field:fields) {
			Autowired autowired = field.getDeclaredAnnotation(Autowired.class);

			if (autowired != null) {
				field.setAccessible(true);
				field.set(instanceBean, getBean(field.getName()));
			}
		}

		// 初始化  init-method

		if (earlySingletonObjects.containsKey(beanName)) {
			instanceBean = earlySingletonObjects.get(beanName);
		}

		// 放到一级缓存中
		singletonObjects.put(beanName, instanceBean);

		// remove二级缓存和三级缓存

		return instanceBean;

	}

	private static Object getSingletonBean(String beanName) {

		Object bean = singletonObjects.get(beanName);
		// 一级缓存没有就说明是循环依赖
		if (bean == null && singletonsCurrentlyInCreation.contains(beanName)) {
			bean = earlySingletonObjects.get(beanName);
			// 从二级缓存拿
			if (earlySingletonObjects.containsKey(beanName)) {
				return bean;
			}
            // 如果二级缓存没有，就从三级缓存拿
			ObjectFactory factory = singletonFactories.get(beanName);

			if (factory != null) {
				earlySingletonObjects.put(beanName, factory.getObject());
			}
		}

		return bean;
	}
}
