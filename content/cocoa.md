---
title: "CoCoALib"
date: 2023-04-06T22:54:01+02:00
---

A GPL C++ library for doing Computations in Commutative Algebra, see http://cocoa.dima.unige.it/cocoalib/.

## Installation

Download and unpack the tar-archive from the website and follow the instructions in doc/txt/INSTALL.txt.

## Example Code

To construct Q(Sqrt(2),Sqrt(1+i)) as an extension field of Q, we first construct Q(Sqrt(1+i)) and then Q(Sqrt(2),Sqrt(1+i)). Finally we show that i is an element of Q(Sqrt(2),Sqrt(1+i)).

```code
#include<iostream>

#include<CoCoA/library.H>

void example() {
	std::cout << "Construction of Q(Sqrt(2),Sqrt(1+i))/Q:" << std::endl;
	auto Q = CoCoA::RingQQ();
	std::cout << "Q = " << Q << std::endl;

	auto Q_PolyRing_x = CoCoA::NewPolyRing(Q, CoCoA::symbols("xx")); // single symbols cause an error
	auto x = CoCoA::indets(Q_PolyRing_x)[0];
	std::cout << "Q[x] = " << Q_PolyRing_x << std::endl;
	auto p_x = (x*x+1)*(x*x+1)+1;
	std::cout << "Factorisation of " << p_x << " over Q: ";
	std::cout << CoCoA::factor(p_x) << std::endl;
	auto Qx = CoCoA::NewQuotientRing(Q_PolyRing_x, CoCoA::ideal(p_x));
	std::cout << "Q(x) = " << Qx << std::endl;
	//auto Hom_Qx = CoCoA::QuotientingHom(Qx);

	auto Qx_PolyRing_y = CoCoA::NewPolyRing(Qx, CoCoA::symbols("yy"));
	//auto Hom_Qx_PolyRing_y = CoCoA::CoeffEmbeddingHom(Qx_PolyRing_y);
	auto y = CoCoA::indets(Qx_PolyRing_y)[0];
	std::cout << "Q(x)[y] = " << Qx_PolyRing_y << std::endl;
	auto p_y = y*y-2;
	std::cout << "Factorisation of " << p_y << " over Q(x): ";
	std::cout << CoCoA::factor(p_y) << std::endl;
	auto Qxy = CoCoA::NewQuotientRing(Qx_PolyRing_y, CoCoA::ideal(p_y));
	std::cout << "Q(x,y) = " << Qxy << std::endl;
	//auto Hom_Qxy = CoCoA::QuotientingHom(Qxy);

	auto Qxy_PolyRing_z = CoCoA::NewPolyRing(Qxy, CoCoA::symbols("zz"));
	//auto Hom_Qxy_PolyRing_z = CoCoA::CoeffEmbeddingHom(Qxy_PolyRing_z);
	auto z = CoCoA::indets(Qxy_PolyRing_z)[0];
	auto p_z = z*z+1;
	std::cout << "Factorisation of " << p_z << " over Q(x,y): ";
	std::cout << CoCoA::factor(p_z) << std::endl;
	/*auto p2_z = Hom_Qxy_PolyRing_z(Hom_Qxy(Hom_Qx_PolyRing_y(Hom_Qx(x))))*(z*z+1);
	std::cout << "Factorisation of " << p2_z << " over Q(x,y): ";
	std::cout << CoCoA::factor(p2_z) << std::endl;*/
}

int main() {
	CoCoA::GlobalManager CoCoAFoundations;
	try {
		example();
	} catch (const CoCoA::ErrorInfo& err) {
		std::cout << err << std::endl;
		return 1;
	}
	return 0;
}

```

Compile the code via

```bash
g++ -I <path>/CoCoALib-<version>/include -L <path>/CoCoALib-<version>/lib example.cpp -lcocoa -lgmp
```